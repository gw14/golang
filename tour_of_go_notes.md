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
https://go.dev/tour/basics/4
