https://go.dev/tour/basics/1

* every go program is made of packages.

*  programms start running in package “main”.

```go
package main

import (
	fmt
	math/rand
)
func main() {
	fmt.Println(“my favourite number is:”, rand.Intn(10))
}
```
* importing packages can be divided into several import statements:
```
import "fmt"
import "math/rand"
```
* exportes names (that can be used outside of their package) **need** to be capitalized:
  ```
  Pizza #exported
  pizza #not exported
  ``` 
```
x int, y int <=> x, y int
```

* A function can return any number of results.
* A var statement can be at package or function level.

* type casting - explicit
```
i := 42
f := float64(i)
u := uint(f)
```
* const decleration `const Pi = 3.14`

https://go.dev/tour/flowcontrol/1
