```
package main

import "fmt"

func main() {
    arr := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    fmt.Println("The def of arr is: arr := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}")
    fmt.Println("The len of arr ", len(arr))
    fmt.Println("The cap of arr ", cap(arr))
    sli := arr[2:4]
    fmt.Println(sli)
    fmt.Println("The def of sli is:sli := arr[2:4]")
    fmt.Println("The len of sil ", len(sli))
    fmt.Println("The cap of sli ", cap(sli))
```
输出
```
The def of arr is: arr := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
The len of arr  10
The cap of arr  10
[2 3]
The def of sli is:sli := arr[2:4]
The len of sil  2
The cap of sli  8
```
