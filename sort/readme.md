

[golang-sort官方文档](https://golang.org/src/sort/sort.go)
```
// IntSlice attaches the methods of Interface to []int, sorting in increasing order.
type IntSlice []int

func (p IntSlice) Len() int           { return len(p) }
func (p IntSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p IntSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

// Sort is a convenience method.
func (p IntSlice) Sort() { Sort(p) }

// Float64Slice attaches the methods of Interface to []float64, sorting in increasing order
// (not-a-number values are treated as less than other values).
type Float64Slice []float64

func (p Float64Slice) Len() int           { return len(p) }
func (p Float64Slice) Less(i, j int) bool { return p[i] < p[j] || isNaN(p[i]) && !isNaN(p[j]) }
func (p Float64Slice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

// isNaN is a copy of math.IsNaN to avoid a dependency on the math package.
func isNaN(f float64) bool {
	return f != f
}

// Sort is a convenience method.
func (p Float64Slice) Sort() { Sort(p) }

// StringSlice attaches the methods of Interface to []string, sorting in increasing order.
type StringSlice []string

func (p StringSlice) Len() int           { return len(p) }
func (p StringSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p StringSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

// Sort is a convenience method.
func (p StringSlice) Sort() { Sort(p) }
```


[golang-sort经典博文](https://itimetraveler.github.io/2016/09/07/%E3%80%90Go%E8%AF%AD%E8%A8%80%E3%80%91%E5%9F%BA%E6%9C%AC%E7%B1%BB%E5%9E%8B%E6%8E%92%E5%BA%8F%E5%92%8C%20slice%20%E6%8E%92%E5%BA%8F/)
```
package main
 
import (
    "fmt"
    "sort"
)
 
type Person struct {
    Name string
    Age  int
}
 
type PersonWrapper struct {
    people [] Person
    by func(p, q * Person) bool
}
 
type SortBy func(p, q *Person) bool
 
func (pw PersonWrapper) Len() int {    		// 重写 Len() 方法
    return len(pw.people)
}
func (pw PersonWrapper) Swap(i, j int){         // 重写 Swap() 方法
    pw.people[i], pw.people[j] = pw.people[j], pw.people[i]
}
func (pw PersonWrapper) Less(i, j int) bool {    // 重写 Less() 方法
    return pw.by(&pw.people[i], &pw.people[j])
}
 
// 封装成 SortPerson 方法
func SortPerson(people [] Person, by SortBy){
    sort.Sort(PersonWrapper{people, by})
}
 
func main() {
    people := [] Person{
        {"zhang san", 12},
        {"li si", 30},
        {"wang wu", 52},
        {"zhao liu", 26},
    }
 
    fmt.Println(people)
 
    sort.Sort(PersonWrapper{people, func (p, q *Person) bool {
        return q.Age < p.Age    // Age 递减排序
    }})
 
    fmt.Println(people)
 
    SortPerson(people, func (p, q *Person) bool {
        return p.Name < q.Name    // Name 递增排序
    })
 
    fmt.Println(people)
 
}

```
