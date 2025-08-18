https://go.dev/tour/basics/1

*.  every go program is made of packages.

*.  programms start running in package “main”.

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
https://go.dev/tour/basics/4
