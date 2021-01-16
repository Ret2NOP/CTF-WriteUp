# Cyberpunk 1977

## Flag 
`FLAG{ＲE𝖠𝗟_FⅬ𝗔Ｇ}`

## 解題過程

在題目首頁 (http://eofqual.ais3.org:1977/) 上方的 Hint 點進去。

![Cyberpunk 1977 Hint](https://github.com/Ret2NOP/CTF-WriteUp/blob/master/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-hint.png)

點進去會發現網址內含 ?file=hint.txt ，而且下方有 Hint ([VISUAL BASIC 2077](https://drive.google.com/file/d/1AXEcUSwGlBON_1abv919AIw5CSX2I5VU/view))

![Cyberpunk 1977 Hint Page](https://github.com/Ret2NOP/CTF-WriteUp/blob/master/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-hint-page.png)

然後在 Hint 頁面也提到了使用 `tiangolo/uwsgi-nginx-flask` Docker image，代表著後端是用 Python + Flask 架設的，

先來處理 ?file= 的部分，這樣寫通常都會有 LFI 的問題，所以就先嘗試 `/etc/passwd`

![Cyberpunk 1977 LFI etc passwd](https://github.com/Ret2NOP/CTF-WriteUp/blob/master/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-etc-passwd.png)

成功取得！代表有 LFI 的漏洞可以戳 ヽ(●´∀`●)ﾉ，接下來嘗試取得後端原始碼 (通常是 main.py / app.py 之類的檔名)

![Cyberpunk 1977 LFI denied py](https://github.com/Ret2NOP/CTF-WriteUp/blob/master/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-denied-py.png)

喔不 。･ﾟ･(つд`ﾟ)･ﾟ･ ，後端有擋掉以 .py 路徑，可是沒關係ヽ(́◕◞౪◟◕‵)ﾉ
根據過往經驗 Python 會自己產生 .pyc 檔案並放置於 __pycache__ 目錄下以加快執行速度，而 .pyc 檔名是有規格性的而且是可以被反編譯回接近原始碼的檔案。

檔名格式 : `{檔案名稱}.{編譯器}-{版本}.pyc` 例如 : main.cpython-38.pyc

![Cyberpunk 1977 PYC Files](https://github.com/Ret2NOP/CTF-WriteUp/blob/master/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-pyc-files.png)

我先嘗試用 Docker image 最新版本 (3.8) 進行測試，一開始以 app.py (__pycache__/app.cpython-38.pyc) 做為測試，結果失敗，改成 main.py (__pycache__/main.cpython-38.pyc) 為目標後就成功取得 pyc 檔案了 OwO

![Cyberpunk 1977 Wget PYC](https://github.com/Ret2NOP/CTF-WriteUp/blob/master/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-wget-pyc.png)

接下來使用 `uncompyle6` 之類的工具將 pyc 轉回 py 檔案，過程就不贅述了，轉換完後開始讀 Code，歸類出幾項規則

- 1. SQL 語法直接用 format string 進行串接，有 SQL Injection 漏洞
- 2. 輸入的帳號密碼必須要和從資料庫取出的帳號密碼一樣
- 3. 要取得 Flag 的話 Session 內的 is_admin 必須為 True
- 4. 要取得 Flag Token 必須是 `ADMIN-E864E8E8F230374AA7B3B0CE441E209A`
- 5. 若非 admin 使用 `ADMIN` Token 會被阻止
- 6. 在輸出 Flag 的時候 Username 直接和要被 Format 字串串接，有漏洞可以是用傳進來的 Flag 取得其他資料

![Cyberpunk 1977 Wget PYC](https://github.com/Ret2NOP/CTF-WriteUp/blob/master/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-flag-code.png)
![Cyberpunk 1977 Wget PYC](https://github.com/Ret2NOP/CTF-WriteUp/blob/master/2020/AIS3%20EOF%20CTF/assets/Cyberpunk%201977-login-code.png)

接下來要來處理這幾項規則

### SQL Injection

### Token

### Format string

### Session

### Flag!
