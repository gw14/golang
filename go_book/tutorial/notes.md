go run file.go - compile file.go into an executable and run it.
go build file.go - only compile file.go
natively uses UNICODE
go code is organized in packages (libraries or modules in other languages)

fmt.Println(“string”) - function to output “string” to screen (stdout)
fmt is a standard library package, in order to use it’s functions, it need to be imported:
` import “fmt” `

Package main defines the entry point of a program.
inside there’s the function main: ` func main() ` that shows the programs starting point.

package -> import -> declarations (datatypes,functions)
function declaration:	` func Func_name( params ) `
	
program parameters are in OS package:
```go
import “os”

… os.Args
// os.Args[0] is the program name
// os.Args[1:len(os.Args)] , os.Args[1:] are the parameters
```

in strings: operator “+” concatenates.
*for* is the only loop in go:
```go
for initialization ; condition ; post {
//…….
}
```
“_” is the blank identifier, it is assigned values that need to be discarded.

---stopped at page 9---
