# WTF

Author : `Sciuridae` Team : `0xdeadbeef` Flag : `FLAG{𖥂𖢐𖥑𖣠𖤐𖤐𖤐𖣠𖡨𖥶𖦂}`

## 解題過程

根據題目的 Source code 來看，運作流程如下

- 1. 傳入檔案
- 2. 若 POST Data 有包含 log 則當成輸出檔案名稱，若沒有則使用現在時間.log
- 3. 執行 file 指令
- 4. 將結果轉換成 HTML Entities
- 5. // 若檔名第一第二位 != ph 則存檔，若包含則回傳 NO!

![WTF Source](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/WTF-Source.png)

首先就是要找到可以讓 file 輸出我們自己可以控制的檔案格式，我們找到的是圖片檔案 + EXIF 資訊，並將資料往 Comment 塞，我們使用 exiftool 將 Payload 塞進去檔案。
![WTF EXIF](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/WTF-EXIFTool.png)

這是可愛ㄉ圖片給你看看 (ゝ∀･)
![WTF OwO](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/WTF-OwO.png)

接下來要解決會把結果轉換成 HTML Entities 導致 PHP Code 無法執行的問題，在此我們在 log 欄位用了 `php://filter/write` ，搭配上 base64-decode 將結果轉成我們想要的結果，而檔名的部分則利用了路徑處理的特性繞過檢查 d(`･∀･)b

Payload : `php://filter/write=string.strip_tags|convert.base64-decode/resource=owo3.php/.`

然後就可以在 WTF 首頁看到我們剛剛上傳的檔案了

![WTF Logs](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/WTF-logs.png)

直接點點進去把 Query params 加上 peko= 就可以執行指令並取得 Flag ！

![WTF ls](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/WTF-ls.png)
![WTF flag](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/WTF-Flag.png)
