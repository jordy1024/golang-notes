# Golang 中的json序列化与反序列化透析与问题总结

## Golang 官方的json encode
```
func Marshal(v interface{}) ([]byte, error)
Marshal returns the JSON encoding of v.

Marshal traverses the value v recursively. If an encountered value implements the Marshaler interface and is not a nil pointer, Marshal calls its MarshalJSON method to produce JSON. If no MarshalJSON method is present but the value implements encoding.TextMarshaler instead, Marshal calls its MarshalText method and encodes the result as a JSON string. The nil pointer exception is not strictly necessary but mimics a similar, necessary exception in the behavior of UnmarshalJSON.

Otherwise, Marshal uses the following type-dependent default encodings:

Boolean values encode as JSON booleans.

Floating point, integer, and Number values encode as JSON numbers.

String values encode as JSON strings coerced to valid UTF-8, replacing invalid bytes with the Unicode replacement rune. So that the JSON will be safe to embed inside HTML <script> tags, the string is encoded using HTMLEscape, which replaces "<", ">", "&", U+2028, and U+2029 are escaped to "\u003c","\u003e", "\u0026", "\u2028", and "\u2029". This replacement can be disabled when using an Encoder, by calling SetEscapeHTML(false).

Array and slice values encode as JSON arrays, except that []byte encodes as a base64-encoded string, and a nil slice encodes as the null JSON value.

Struct values encode as JSON objects. Each exported struct field becomes a member of the object, using the field name as the object key, unless the field is omitted for one of the reasons given below.

The encoding of each struct field can be customized by the format string stored under the "json" key in the struct field's tag. The format string gives the name of the field, possibly followed by a comma-separated list of options. The name may be empty in order to specify options without overriding the default field name.

The "omitempty" option specifies that the field should be omitted from the encoding if the field has an empty value, defined as false, 0, a nil pointer, a nil interface value, and any empty array, slice, map, or string.

As a special case, if the field tag is "-", the field is always omitted. Note that a field with name "-" can still be generated using the tag "-,".
```


func Marshal(v interface{}) ([]byte, error)

https://golang.org/pkg/encoding/json/#Marshal
https://g  ithub.com/json-iterator/go
http://vearne.cc/archives/433
https://javasgl.github.io/go-json-iterator/
http://jsoniter.com/index.cn.html
http://xiaorui.cc/archives/5108
https://blog.csdn.net/wangshubo1989/article/details/78709802
https://blog.csdn.net/H517604180/article/details/83307978
https://zhuanlan.zhihu.com/p/111733692
http://www.hatlonely.com/2018/01/28/golang-json-%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90/index.html
https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%88%97%E8%A1%A8
https://studygolang.com/articles/19721
https://studygolang.com/articles/6050
https://hacpai.com/article/1524558037151

### 问题记录
- 依赖Json内容统计长度
- 依赖Json的内容做加解密等等操作
- unicode编码资源  https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%88%97%E8%A1%A8



## Golang 官方的json decode


## 第三方的库
### 滴滴出行开源的json库
https://github.com/json-iterator/go 



## 性能比对与原理透析 
### encodemap结构时的说明


## 附录
https://golang.org/pkg/encoding/json/#Marshal
https://github.com/json-iterator/go
http://vearne.cc/archives/433
https://javasgl.github.io/go-json-iterator/
http://jsoniter.com/index.cn.html
http://xiaorui.cc/archives/5108
https://blog.csdn.net/wangshubo1989/article/details/78709802
https://blog.csdn.net/H517604180/article/details/83307978
https://zhuanlan.zhihu.com/p/111733692
http://www.hatlonely.com/2018/01/28/golang-json-%E6%80%A7%E8%83%BD%E5%88%86%E6%9E%90/index.html
https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%88%97%E8%A1%A8
https://studygolang.com/articles/19721
https://studygolang.com/articles/6050
https://hacpai.com/article/1524558037151


