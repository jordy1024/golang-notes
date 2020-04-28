[base64原理笔记]
#### base64编码原理
```
```

#### base64_urlsafe编码原理
```
```

#### golang 中的base64包及其函数
```
https://golang.org/pkg/encoding/base64/

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
safeurl := strings.Replace(cryptedByteStr, "_", "/", -1)
safeurl = strings.Replace(safeurl, "-", "+", -1)
bstr, _ := base64.RawStdEncoding.DecodeString(safeurl)
```

#### golang官方base64文档翻译
```
https://golang.org/pkg/encoding/base64/
```



