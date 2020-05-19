## 逃逸分析  


```
go build -gcflags '-m -l' main.go
加-l是为了不让foo函数被内联。
```

