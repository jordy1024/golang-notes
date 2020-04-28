## [Aes encrypt and decrypt and base64UrslSafe]

```
前提是要深刻理解base64编码以及base64-urlsafe编码的原理
```
```
    str := "xxxxx"
    key := "1111111111111111"
    fmt.Println("加密前原串", str)

    cryptedByteStr := util.AesEncryptV2(str, key)
    fmt.Println("AesEncryptV2加密后的串", cryptedByteStr)

    safeurl := strings.Replace(cryptedByteStr, "_", "/", -1)
    safeurl = strings.Replace(safeurl, "-", "+", -1)
    bstr, _ := base64.RawStdEncoding.DecodeString(safeurl)
    jmstr := util.AesDecrypt(bstr, []byte(key))
    fmt.Println("解密后原串", string(jmstr))
```

```
func Base64UrlSafeEncode(source []byte) string {
    // Base64 Url Safe is the same as Base64 but does not contain '/' and '+' (replaced by '_' and '-') and trailing '=' are removed.
    bytearr := base64.StdEncoding.EncodeToString(source)
    safeurl := strings.Replace(string(bytearr), "/", "_", -1)
    safeurl = strings.Replace(safeurl, "+", "-", -1)
    safeurl = strings.Replace(safeurl, "=", "", -1)
    return safeurl
}
```
```
func AesEncrypt(src, key string) []byte {
    block, err := aes.NewCipher([]byte(key))
    if err != nil {
        //fmt.Println("key error1", err)
        BhAlarm(BH_LOG_DEFAULT, err, "AesEncrypt NewCipher")
    }
    if src == "" {
        //fmt.Println("plain content empty")
        BhAlarm(BH_LOG_DEFAULT, err, "AesEncrypt src: plain content empty")
    }
    ecb := NewECBEncrypter(block)
    content := []byte(src)
    content = PKCS5Padding(content, block.BlockSize())
    crypted := make([]byte, len(content))
    ecb.CryptBlocks(crypted, content)
    // 普通base64编码加密 区别于urlsafe base64
    //fmt.Println("base64 result:", base64.StdEncoding.EncodeToString(crypted))

    //fmt.Println("base64UrlSafe result:", Base64UrlSafeEncode(crypted))
    return crypted
}
```

```
func AesEncryptV2(src, key string) string {
    block, err := aes.NewCipher([]byte(key))
    if err != nil {
        //fmt.Println("key error1", err)
        BhAlarm(BH_LOG_DEFAULT, err, "AesEncrypt NewCipher")
    }
    if src == "" {
        //fmt.Println("plain content empty")
        BhAlarm(BH_LOG_DEFAULT, err, "AesEncrypt src: plain content empty")
    }
    ecb := NewECBEncrypter(block)
    content := []byte(src)
    content = PKCS5Padding(content, block.BlockSize())
    crypted := make([]byte, len(content))
    ecb.CryptBlocks(crypted, content)
    // 普通base64编码加密 区别于urlsafe base64
    //fmt.Println("base64 result:", base64.StdEncoding.EncodeToString(crypted))

    //fmt.Println("base64UrlSafe result:", Base64UrlSafeEncode(crypted))
    return Base64UrlSafeEncode(crypted)
}
```


```
func AesDecrypt(crypted, key []byte) []byte {
    block, err := aes.NewCipher(key)
    if err != nil {
        //fmt.Println("err is:", err)
        BhAlarm(BH_LOG_DEFAULT, err, "AesDecrypt NewCipher")
    }
    blockMode := NewECBDecrypter(block)
    origData := make([]byte, len(crypted))
    blockMode.CryptBlocks(origData, crypted)
    origData = PKCS5UnPadding(origData)
    //fmt.Println("source is :", origData, string(origData))
    return origData
}
```


