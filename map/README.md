```
package main

import "fmt"

func main() {
    m := make(map[string]int, 3)
    m["a"] = 1
    m["b"] = 2
    m["c"] = 3
    m["d"] = 4//在这步骤的时候就发生自动扩容了？如何验证？？
    fmt.Println(len(m))
}
```
