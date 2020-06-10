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



## 切片精华  

- 切片slice原理    
每个切片的本质都是一个结构体，有一个指针指向一个底层数组；   
加入多个切片都指向同一个数组，则需谨慎修改切片内容，以防影响其他切片；   
额外，每个切片都保存了当前切片的长度和底层数组的容量，所以可以直接通过调用len和cap获取切片的长度和容量。   
时间复杂度O(1)；   
通过函数传递切片时，不会拷贝整个切片，因为切片本身只是个结构体而已
```
type slice struct {   
    array unsafe.Pointer   
    len   int
    cap   int
}
``` 

   


- 切片定义    
make 创建切片时可根据实际需要预分配容量，尽量避免追加过程中扩容操作，有利于提升性能；   


- 切片扩容 append   
扩容策略跟切片目前大小有关系；   
使用append()向切片追加元素时有可能触发扩容，扩容后将会生成新的切片   
所以创建切片时可根据实际需要预分配容量，尽量避免追加过程中扩容操作，有利于提升性能；    


- 切片拷贝 copy   
copy的元素个数取决于两个切片长度的最小值 。   
即copy过程中不会发生扩容。      

