
# AA公司BC平臺介面文檔 v3.2.0

## 1 規範說明

### 1.1 通信協議

HTTPS協議

### 1.2 請求方法
所有介面只支援POST方法發起請求。

### 1.3 字元編碼
HTTP通訊及報文BASE64編碼均採用UTF-8字元集編碼格式。

### 1.4 格式說明
元素出現要求說明：

符號				|說明
:----:			|:---
R				|報文中該元素必須出現（Required）
O				|報文中該元素可選出現（Optional）
C				|報文中該元素在一定條件下出現（Conditional）

### 1.5 報文規範說明

1. 報文規範僅針對交易請求資料進行描述；  

2. 報文規範中請求報文的內容為Https請求報文中RequestData值的明文內容；

3. 報文規範分為請求報文和回應報文。請求報文描述由發起方，回應報文由報文接收方回應。

### 1.6 請求報文結構
介面只接收兩個參數 **RequestData** 和 **SignData** ，其中RequestData的值為請求內容，SignData的值為簽名內容。

#### 1.6.1 參數說明
**RequestData（請求內容）：** 其明文為每次請求的具體參數，採用 JSON 格式，依次經過 DES 加密（以UTF-8編碼、BASE64編碼輸出結果）和 URLEncode 後，作為 RequestData 的值。  

**SignData（簽名內容）：** 請求參數（明文）的MD5加密字串，用於校驗RequestData是否合法。

#### 1.6.2 請求內容（RequestData）明文結構說明

採用JSON格式，其中包含Header（公有參數）、Body（私有參數）節點：

名稱		|描述											|備註
:--		|:--											|:--
公共參數	|每個介面都包含的通用參數，以JSON格式存放在Header屬性	|詳見以下公共參數說明
私有參數	|每個介面特有的參數，以JSON格式存放在Body屬性		|詳見每個介面定義

**公共參數說明：**

公共參數（Header）是用於標識產品及介面鑒權的參數，每次請求均需要攜帶這些參數：

參數名稱				|類型		|出現要求	|描述  
:----				|:---		|:------	|:---	
Token				|string		|R			|用戶登錄後token，沒有登錄則為空字串
Version				|string		|R			|介面版本號
SystemId			|int		|R			|機構號，請求的系統Id
Timestamp			|long		|R			|當前UNIX時間戳記


#### 1.6.3 校驗流程：
服務端接收到請求後首先對RequestData進行DES解密出JSON字串，然後對JSON字串進行MD5加密，加密後的值與請求中的SignData值進行對比，如對比通過，視為合法請求，否則視為非法請求。

**DES加密/解密函數示例：**

C#版：

```
/// <summary>
/// 進行DES加密。
/// </summary>
/// <param name="decryptString">要加密的字串。</param>
/// <param name="secretKey">金鑰，且必須為8位。</param>
/// <returns>以Base64格式返回的加密字串。</returns>
public static string DesEncrypt(string decryptString, string secretKey)
{
    using (DESCryptoServiceProvider des = new DESCryptoServiceProvider())
    {
        byte[] inputByteArray = Encoding.UTF8.GetBytes(decryptString);
        des.Key = Encoding.ASCII.GetBytes(secretKey);
        des.IV = Encoding.ASCII.GetBytes(secretKey);
        MemoryStream ms = new MemoryStream();
        using (CryptoStream cs = new CryptoStream(ms, des.CreateEncryptor(), CryptoStreamMode.Write))
        {
            cs.Write(inputByteArray, 0, inputByteArray.Length);
            cs.FlushFinalBlock();
            cs.Close();
        }
        string str = Convert.ToBase64String(ms.ToArray());
        ms.Close();
        return str;
    }
}

/// <summary>
/// 進行DES解密。
/// </summary>
/// <param name="encryptedString">要解密的以Base64</param>
/// <param name="secretKey">金鑰，且必須為8位。</param>
/// <returns>已解密的字串。</returns>
public static string DesDecrypt(string encryptedString, string secretKey)
{
    byte[] inputByteArray = Convert.FromBase64String(encryptedString);
    using (DESCryptoServiceProvider des = new DESCryptoServiceProvider())
    {
        des.Key = Encoding.ASCII.GetBytes(secretKey);
        des.IV = Encoding.ASCII.GetBytes(secretKey);
        MemoryStream ms = new MemoryStream();
        using (CryptoStream cs = new CryptoStream(ms, des.CreateDecryptor(), CryptoStreamMode.Write))
        {
            cs.Write(inputByteArray, 0, inputByteArray.Length);
            cs.FlushFinalBlock();
            cs.Close();
        }
        string str = Encoding.UTF8.GetString(ms.ToArray());
        ms.Close();
        return str;
    }
}
```

JAVA版：

```
/* DES解密 */
public static String decrypt(String message, String key) throws Exception {

    byte[] bytesrc = Base64.decode(message);
    //convertHexString(message);
    Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding");
    DESKeySpec desKeySpec = new DESKeySpec(key.getBytes("UTF-8"));
    SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
    SecretKey secretKey = keyFactory.generateSecret(desKeySpec);
    IvParameterSpec iv = new IvParameterSpec(key.getBytes("UTF-8"));
    cipher.init(Cipher.DECRYPT_MODE, secretKey, iv);
    byte[] retByte = cipher.doFinal(bytesrc);
    return new String(retByte);
}


/* DES加密 */
public static byte[] encrypt(String message, String key) throws Exception {
    Cipher cipher = Cipher.getInstance("DES/CBC/PKCS5Padding");
    DESKeySpec desKeySpec = new DESKeySpec(key.getBytes("UTF-8"));
    SecretKeyFactory keyFactory = SecretKeyFactory.getInstance("DES");
    SecretKey secretKey = keyFactory.generateSecret(desKeySpec);
    IvParameterSpec iv = new IvParameterSpec(key.getBytes("UTF-8"));
    cipher.init(Cipher.ENCRYPT_MODE, secretKey, iv);
    return cipher.doFinal(message.getBytes("UTF-8"));
}
```

#### 1.6.4 DES金鑰

測試環境：az2ih1uY

生產環境：另外提供。

#### 1.6.5 請求報文示例
請求內容明文：

```
{
    "Header":{
        "Token":"2366CF921FAD44CCBB07FF9CD02FC90E",
        "Version":"3.2.0",
        "SystemId":100,
        "Timestamp":1502870664
    },
    "Body":{
        "Mobile":"18520322032",
        "Password":"acb000000"
    }
}

```

請求報文示例：

```
url?RequestData=UFAYIRF21XzGoaAaEU54qoDBYaFkT2KbRpWxKZuqqltApdIneF7AjlEArPLsg3%2Fo1Pu7FHFmsKZn%0A9KJb%2BGuwx0P%2F3jzv2TgwUpVtgwEdfd0vIRfqEF4jCouldaxxVBjbHvd%2F08pUoYJDNZJLvNrJ%2BsK4%0A79de92T0Cyu4hKNMUPtVI7Tp0IC%2BBw%3D%3D&SignData=0865c7d625f90d3bb5457f5d9ac3725d
```

### 1.7 回應報文結構
#### 1.7.1 結構說明
所有介面回應均採用JSON格式，如無特殊說明，每次請求的返回值中，都包含下列欄位：

參數名稱						|類型		|出現要求	|描述  
:----						|:---		|:------	|:---	
Code						|int		|R			|回應碼，代碼定義請見“附錄A 回應嗎說明”
Msg							|string		|R			|回應描述
Data						|object		|R			|每個介面特有的參數，詳見每個介面定義


#### 1.7.2 回應報文示例

```
{
    "Code":200,
    "Msg":"調用成功",
    "Data":{
        "Channel":"A10086",
        "Type":7004
    }
}
```


## 2. 介面定義

### 2.1 密碼登錄
- **介面說明：** 密碼登錄
- **介面位址：** /account/signin

#### 2.1.1 請求參數
  
參數名稱						|類型		|出現要求	|描述  
:----						|:---		|:------	|:---	
Header						|&nbsp;		|R			|請求報文頭
&emsp;Token					|string		|R			|用戶登錄後token，沒有登錄則為空字串
&emsp;Version				|string		|R			|介面版本號
&emsp;SystemId				|int		|R			|機構號，請求的系統Id
&emsp;Timestamp				|long		|R			|當前UNIX時間戳記
Body						|&nbsp;		|R			|&nbsp;
&emsp;Mobile				|string		|R			|手機號
&emsp;Password				|string		|R			|密碼


請求示例：

```
{
    "Header":{
        "Token":"",
        "Version":"3.2.0",
        "SystemId":100,
        "Timestamp":1502870664
    },
    "Body":{
        "Mobile":"18520322032",
        "Password":"acb000000"
    }
}

```


#### 2.1.2 返回結果

參數名稱						|類型		|出現要求	|描述  
:----						|:---		|:------	|:---	
Code						|int		|R			|回應碼，代碼定義請見“附錄A 回應嗎說明”
Msg							|string		|R			|&nbsp;
Data						|object		|R			|&nbsp;
&emsp;UserId				|string		|R			|用戶Id

示例：

```
{
    "Code":200,
    "Msg":"登錄成功",
    "Data":{
        "UserId":"7D916C7283434955A235C17DD9B71C64"
    }
}
```



### 2.2 獲取登錄使用者資訊
- **介面說明：** 獲取登錄使用者資訊
- **介面位址：** /account/profile

#### 2.2.1 請求參數
  
參數名稱						|類型		|出現要求	|描述  
:----						|:---		|:------	|:---	
Header						|&nbsp;		|R			|請求報文頭
&emsp;Token					|string		|R			|用戶登錄後token，沒有登錄則為空字串
&emsp;Version				|string		|R			|介面版本號
&emsp;SystemId				|int		|R			|機構號，請求的系統Id
&emsp;Timestamp				|long		|R			|當前UNIX時間戳記
Body						|&nbsp;		|R			|&nbsp;



請求示例：

```

{
    "Header":{
        "Token":"CA64A439E7C344B0BA7F5C825E17C7AB",
        "Version":"3.2.0",
        "SystemId":100,
        "Timestamp":1502870664
    },
    "Body":null
}

```


#### 2.2.2 返回結果

參數名稱						|類型		|出現要求	|描述  
:----						|:---		|:------	|:---	
Code						|int		|R			|回應碼，代碼定義請見“附錄A 回應嗎說明”
Msg							|string		|R			|&nbsp;
Data						|object		|R			|&nbsp;
&emsp;UserId				|string		|R			|用戶Id
&emsp;RealName				|string		|R			|姓名
&emsp;ImageUrl				|string		|R			|頭像
&emsp;Score					|int		|R			|積分
&emsp;Nickname				|string		|R			|昵稱
&emsp;Sex					|int		|R			|性別：0-未知、1-男、2-女
&emsp;Title					|string		|R			|頭銜


示例：

```
{
    "Code":200,
    "Msg":"處理成功",
    "Data":{
        "UserId":"7D916C7283434955A235C17DD9B71C64",
        "RealName":"張三",
        "ImageUrl":"https://img.xx.net/afdicew8751.png",
        "Score":4732,
        "Nickname":"張冠李戴",
        "Sex":1,
        "Title":"俠客Lv4"
    }
}
```


## 3 附錄A 回應碼說明

回應碼	|說明  
:----	|:---
200		|處理成功
301		|解析報文錯誤
302		|無效調用憑證
303		|參數不正確
500		|系統內部錯誤
999		|處理失敗


## 4 附錄B 幣種

幣種		|說明  
:----	|:---
RMB		|人民幣
HKD		|港幣
JPY		|日元
TWD		|新臺幣
USD		|美元
VND		|越南盾
THB		|泰銖
