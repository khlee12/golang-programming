Get Started

https://go.dev/doc/ 

# [Tutorial: Getting started](https://go.dev/doc/tutorial/getting-started.html)
**A tutorial of short topics introducing functions, error handling, arrays, maps, unit testing, and compiling.**

To enable dependency tracking for your code by creating a go.mod file, run the go mod init command, giving it the name of the module your code will be in. The name is the module's module path.
```
$ go mod init example/hello
go: creating new go.mod: module example/hello
```
-> 会生成一个go.mod文件，并定义上面生成的module名。

所以这里的example/并不是指某个dir
```
module example/hello

go 1.19
```
-> Hello world!

```
package main // 声明一个主软件包（软件包是对函数进行分组的一种方式，由同一目录下的所有文件组成）。

import "fmt" // 导入流行的fmt包，其中包含用于格式化文本的函数，包括打印到控制台的函数。该软件包是安装 Go 时获得的标准库软件包之一。
import "rsc.io/quote" // 导入外部包

func main() { // 执行一个main函数，向控制台打印一条信息。运行main程序包时，默认情况下会执行main函数。
    fmt.Println("Hello, World!")
    fmt.Println(quote.Go()) // 执行外部包的函数
}
```

```
go mod tidy
```
- Enable dependency tracking for the code you're about to write.
- 生成一个go.sum文件，用于寻找并下载添加已被验证的模块（默认下载最新版）
- 亦会更行go.mod文件, 将下载的模块作为requirements加到当前包的定义中
```
module example/hello

go 1.19

require rsc.io/quote v1.5.2

require (
	golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c // indirect
	rsc.io/sampler v1.3.0 // indirect
)
```
# [Tutorial: Create a module](https://go.dev/doc/tutorial/create-module.html)

## Start a module that others can use

Go code is grouped into packages, and packages are grouped into modules.

In Go, a function whose name starts with a capital letter can be called by a function not in the same package. This is known in Go as an exported name.

In Go, the := operator is a shortcut for declaring and initializing a variable in one line
```
package greetings // Declare a greetings package to collect related functions.

import "fmt"

func Hello(name string) string {
	message := fmt.Sprintf("Hi, %v. Welcome!", name)
	// Also can be written as:
	// var message string
	// message = fmt.Sprintf("Hi, %v. Welcome!", name)
	return message
}
```
How to call the module:
```
<home>/
 |-- greetings/
 |-- hello/
```
hello/hello.go:
```
package main
import "fmt"
import "example.com/greetings"
// import "rsc.io/quote"

func main(){
	message := greetings.Hello("Victoria")
	fmt.Println(message)
}
```
-> 因为example.com/greetings还没有发布到网上，所以我们不能直接搜索并下载。我们需要用命令将这个包指向我们local定义的包
```
$ go mod edit -replace example.com/greetings=../greetings
```
-> go.mod也发生了变化
```
module example.com/hello

go 1.19

replace example.com/greetings => ../greetings
```
-> 用go mod tidy来“下载”local的包

-> go.mod发生了变化
```
module example.com/hello

go 1.19

replace example.com/greetings => ../greetings

require example.com/greetings v0.0.0-00010101000000-000000000000
```
## Return and handle an error 
- That's common error handling in Go: Return an error as a value so the caller can check for it.
```
import "log"
func main() {
    message, err := greetings.Hello("")
	if err != nil {
		log.Fatal(err) // ここでexitされて↓printされない
	}
	fmt.Println(message)
}
```
-> Fatal is equivalent to Print() followed by a call to os.Exit(1).

## Return a random greeting 

Go slice (Go's dynamically-sized arrays).

A slice is like an array, except that its size changes dynamically as you add and remove items. The slice is one of Go's most useful types.
```
func randomFormat() string {
	formats := []string{ // slice definition
		"Hi, %v. Welcome",
		"Great to see you, %v!",
		"Hail, %v! Well met!",
	}
	return formats[rand.Intn(len(formats))]
}
```
-> When declaring a slice, you omit its size in the brackets, like this: []string. This tells Go that the size of the array underlying the slice can be dynamically changed.

[link](https://blog.y-yuki.net/entry/2017/05/02/000000)

Go言語のfmtパッケージに存在するPrintf関数は[書式](https://golang.org/pkg/fmt/#hdr-Printing)を指定して標準出力に書き込みを行います。
C言語のprintfに良く似ています。が、C言語には存在しない書式がいくつか加わっています。
その中の代表格として以下のような書式が存在しています。

- `%T`
  - 対象データの型情報を埋め込む
- `%v`
  - デフォルトフォーマットで対象データの情報を埋め込む
- `%+v`
  - 構造体を出力する際に、%vの内容にフィールド名も加わる
- `%#v`
  - Go言語のリテラル表現で対象データの情報を埋め込む

## Return greetings for multiple people

key, value pair(map):
```
messages := make(map[string]string)
```
-> make()は組み込み関数の1つで、makeを使用することでsliceを指定することができます！
```
make([]T, len, cap)
```
第一引数の「[]T」はスライスの要素の型を指定し、第二引数の「len」はスライスの長さを指定し、第三引数の「cap」はスライスの容量を指定することができます。
```
func Hellos(names []string) (map[string]string, error) {
	messages := make(map[string]string)
	for _, name := range names {
		message, err := Hello(name)
		if err != nil {
			return nil, err
		}
		messages[name] = message
	}
	return messages, nil
}
```
->  But there's a hitch. Changing the Hello function's parameter from a single name to a set of names would change the function's signature. If you had already published the example.com/greetings module and users had already written code calling Hello, that change would break their programs.

In this situation, a better choice is to write a new function with a different name. The new function will take multiple parameters. That preserves the old function for backward compatibility.

-> 如果要改变参数的type，最好是再做一个新的函数with new parameter datatype

-> 如果想要用module的versioning来实现呢？

-> In this for loop, range returns two values: the index of the current item in the loop and a copy of the item's value

## Add a test

Ending a file's name with _test.go tells the go test command that this file contains test functions.
```
package greetings
// Implement test functions in the same package as the code you're testing.


import (
    "testing"
    "regexp"
)

// TestHelloName calls greetings.Hello with a name, checking
// for a valid return value.
func TestHelloName(t *testing.T) {
    // Test function names have the form Test<Name>, 
    // where Name says something about the specific test
    // Also, test functions take a pointer to the testing package's testing.T type as a parameter. 
    // You use this parameter's methods for reporting and logging from your test.
    name := "Gladys"
    want := regexp.MustCompile(`\b`+name+`\b`)
    msg, err := Hello("Gladys")
    if !want.MatchString(msg) || err != nil {
        t.Fatalf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
    }
}

// TestHelloEmpty calls greetings.Hello with an empty string,
// checking for an error.
func TestHelloEmpty(t *testing.T) {
    msg, err := Hello("")
    if msg != "" || err == nil {
        t.Fatalf(`Hello("") = %q, %v, want "", error`, msg, err)
    }
}
```
The `go test` command executes test functions (whose names begin with `Test`) in test files (whose names end with _test.go). You can add the `-v` flag to get verbose output that lists all of the tests and their results.
```
$ go test
PASS
ok      example.com/greetings   0.364s

$ go test -v
=== RUN   TestHelloName
--- PASS: TestHelloName (0.00s)
=== RUN   TestHelloEmpty
--- PASS: TestHelloEmpty (0.00s)
PASS
ok      example.com/greetings   0.372s
```
## Compile and install the application

The `go build` command compiles the packages, along with their dependencies, but it doesn't install the results.

The `go install` command compiles and installs the packages.

in hello/:
```
go build
```
-> 生成hello文件，可以用 ./hello来执行

-> 如果不想指定path，或者想跟一般pkg一样install之后以一种CLI来执行的话：
```
go list -f '{{.Target}}'
--> discover the install path
export PATH=$PATH:/path/to/your/install/directory
go env -w GOBIN=/path/to/your/bin
go install
```
就可以用 $ hello来执行
```
$ hello
map[Darrin:Hail, Darrin! Well met! Gladys:Great to see you, Gladys! Samantha:Hail, Samantha! Well met!]
```

# [Tutorial: Getting started with multi-module workspaces](https://go.dev/doc/tutorial/workspaces.html)

Introduces the basics of creating and using multi-module workspaces in Go. Multi-module workspaces are useful for making changes across multiple modules.

init workspace
```
# under workspace dir
$ go work init ./hello
```
-> hello is existing dir

-> The `go work init` command tells `go` to create a `go.work` file for a workspace containing the modules in the `./hello` directory.

go.work:
```
go 1.18

use ./hello
```

-> The go directive tells Go which version of Go the file should be interpreted with. It’s similar to the go directive in the go.mod file.

-> The use directive tells Go that the module in the hello directory should be main modules when doing a build.

-> So in any subdirectory of workspace the module will be active.
```
$ go run ./hello
olleH
```
-> **The Go command includes all the modules in the workspace as main modules.**

Add module to the workspace:
```
$ go work use ./example/hello
```
-> go.work updated to:
```
go 1.19

use (
	./example/hello
	./hello
)
```
- `go work use [-r] [dir]` adds a `use` directive to the `go.work` file for `dir`, if it exists, and removes the `use` directory if the argument directory doesn’t exist. The `-r` flag examines subdirectories of `dir` recursively.
- `go work edit` edits the `go.work` file similarly to `go mod edit`
- `go work sync` syncs dependencies from the workspace’s build list into each of the workspace modules.

# [Tutorial: Developing a RESTful API with Go and Gin](https://go.dev/doc/tutorial/web-service-gin.html)

**Introduces the basics of writing a RESTful web service API with Go and the Gin Web Framework.**

Gin is a HTTP web framework written in Go (Golang). It features a Martini-like API, but with performance up to 40 times faster than Martini.

自前のdata typeを作ったあと、それを使って変数の定義
```
// album represents data about a record album.
type album struct {
    ID     string  `json:"id"`
    Title  string  `json:"title"`
    Artist string  `json:"artist"`
    Price  float64 `json:"price"`
}

// albums slice to seed record album data.
var albums = []album{
    {ID: "1", Title: "Blue Train", Artist: "John Coltrane", Price: 56.99},
    {ID: "2", Title: "Jeru", Artist: "Gerry Mulligan", Price: 17.99},
    {ID: "3", Title: "Sarah Vaughan and Clifford Brown", Artist: "Sarah Vaughan", Price: 39.99},
}
```

-> sliceの強みがわかる。api経由で新しいデータ挿入時、arr sizeを気にしなくてOK

-> もちろん一般的にはdbとinteractするが、今回みたいにarrをmemoryに乗せて使う場合にはsliceがsimpleで使いやすい

GET
```
// getAlbums responds with the list of all albums as JSON.
func getAlbums(c *gin.Context) {
    c.IndentedJSON(http.StatusOK, albums)
}
```
-> `gin.Context` is the most important part of Gin. It carries request details, validates and serializes JSON, and more. (Despite the similar name, this is different from Go’s built-in `context` package.)

-> Note that you can replace `Context.IndentedJSON` with a call to `Context.JSON` to send more compact JSON. In practice, the indented form is much easier to work with when debugging and the size difference is usually small.
```
func main() {
    router := gin.Default()
    router.GET("/albums", getAlbums)

    router.Run("localhost:8080")
}
```
```
go get .
```
-> get dependencies for code in the current directory

POST
```
// postAlbums adds an album from JSON received in the request body.
func postAlbums(c *gin.Context) {
    var newAlbum album

    // Call BindJSON to bind the received JSON to
    // newAlbum.
    if err := c.BindJSON(&newAlbum); err != nil {
        return
    }

    // Add the new album to the slice.
    albums = append(albums, newAlbum)
    c.IndentedJSON(http.StatusCreated, newAlbum)
}
```
main.goも一緒に更新
```
func main() {
	router := gin.Default()
	router.GET("/albums", getAlbums)
	router.POST("/albums", postAlbums)
	router.Run("localhost:8080")
}
```
```
$ curl http://localhost:8080/albums \
    --include \
    --header "Content-Type: application/json" \
    --request "POST" \
    --data '{"id": "4","title": "The Modern Sound of Betty Carter","artist": "Betty Carter","price": 49.99}'
```
GET with params
```
// getAlbumByID locates the album whose ID value matches the id
// parameter sent by the client, then returns that album as a response.
func getAlbumByID(c *gin.Context) {
    id := c.Param("id")

    // Loop over the list of albums, looking for
    // an album whose ID value matches the parameter.
    for _, a := range albums {
        if a.ID == id {
            c.IndentedJSON(http.StatusOK, a)
            return
        }
    }
    c.IndentedJSON(http.StatusNotFound, gin.H{"message": "album not found"})
}
```
main.go:
```
router.GET("/albums/:id", getAlbumByID)
```
-> In Gin, the colon preceding an item in the path signifies that the item is a path parameter.

# [Tutorial: Getting started with generics](https://go.dev/doc/tutorial/generics.html)

**With generics, you can declare and use functions or types that are written to work with any of a set of types provided by calling code.**
```
package main
```
-> A standalone program (as opposed to a library) is always in package main.

generic function to handle multiple types:
```
// SumIntsOrFloats sums the values of map m. It supports both int64 and float64
// as types for map values.
func SumIntsOrFloats[K comparable, V int64 | float64](m map[K]V) V {
    var s V
    for _, v := range m {
        s += v
    }
    return s
}
```
-> To support values of either type, that single function will need a way to declare what types it supports. Calling code, on the other hand, will need a way to specify whether it is calling with an integer or float map.

-> To support this, you’ll write a function that declares type parameters in addition to its ordinary function parameters. These type parameters make the function generic, enabling it to work with arguments of different types. You’ll call the function with type arguments and ordinary function arguments.

-> Each type parameter has a type constraint that acts as a kind of meta-type for the type parameter. Each type constraint specifies the permissible type arguments that calling code can use for the respective type parameter.

-> While a type parameter’s constraint typically represents a set of types, at compile time the type parameter stands for a single type – the type provided as a type argument by the calling code. If the type argument’s type isn’t allowed by the type parameter’s constraint, the code won’t compile.

-> Specify for the `K` type parameter the type constraint `comparable`. Intended specifically for cases like these, the `comparable` constraint is predeclared in Go. It allows any type whose values may be used as an operand of the comparison operators `==` and `!=`. Go requires that map keys be comparable. So declaring `K` as `comparable` is necessary so you can use `K` as the key in the map variable. It also ensures that calling code uses an allowable type for map keys.

-> Specify for the `V` type parameter a constraint that is a union of two types: `int64` and `float64`. Using `|` specifies a union of the two types, meaning that this constraint allows either type. Either type will be permitted by the compiler as an argument in the calling code.

-> Note that we know `map[K]V` is a valid map type because `K` is a comparable type. If we hadn’t declared `K` comparable, the compiler would reject the reference to `map[K]V`.

main.go
```
fmt.Printf("Generic Sums: %v and %v\n",
    SumIntsOrFloats[string, int64](ints),
    SumIntsOrFloats[string, float64](floats))
```
-> In calling the generic function you wrote, you specified type arguments that told the compiler what types to use in place of the function’s type parameters. As you’ll see in the next section, in many cases you can omit these type arguments because the compiler can infer them.

omit the type parameters also works:
```
fmt.Printf("Generic Sums, type parameters inferred: %v and %v\n",
    SumIntsOrFloats(ints),
    SumIntsOrFloats(floats))
```
-> Note that this isn’t always possible. For example, if you needed to call a **generic function that had no arguments**, you would **need to include the type arguments** in the function call.

use Constrains to refine code:
```
type Number interface {
    int64 | float64
}
func SumNumbers [K comparable, V Number] (m map[K]V) V {
	var s V
	for _, v := range m {
		s += v
	}
	return s
}

// main func...
fmt.Printf("Generic Sums with constraint: %v and %v \n", SumNumbers(m1), SumNumbers(m2))
```
# [Tutorial: Getting started with fuzzing](https://go.dev/doc/tutorial/fuzz.html)

**Fuzzing can generate inputs to your tests that can catch edge cases and security issues that you may have missed.**

main.go
```
package main

import "fmt"

func main() {
	input := "The quick brown fox jumped over the lazy dog"
	rev := Reverse(input)
	doubleRev := Reverse(rev)
	fmt.Printf("original: %q\n", input)
    fmt.Printf("reversed: %q\n", rev)
    fmt.Printf("reversed again: %q\n", doubleRev)

}

func Reverse(s string) string {
	b := []byte(s)
	fmt.Printf("b: %v", b)
	for i, j := 0, len(b)-1; i < len(b)/2; i, j = i+1, j-1 {
		b[i], b[j] = b[j], b[i]
	}
	return string(b)
}
```
add unit test
```
package main

import (
	"testing"
)

func TestReverse(t *testing.T) {
	testcases:= []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
        {" ", " "},
        {"!12345", "54321!"},
	}
	for _, tc := range testcases {
		actual := Reverse(tc.in)
		if actual != tc.want {
			t.Errorf("Reverse: %q, want %q", actual, tc.want)
		}
	}
}
```
-> The unit test has limitations, namely that each input must be added to the test by the developer. One benefit of fuzzing is that it comes up with inputs for your code, and may identify edge cases that the test cases you came up with didn’t reach.

add fuzz test
```
func FuzzReverse(f *testing.F) {
	testcases := []string{"Hello, world", " ", "!12345"} // -> inputのみ定義している
	for _, tc := range testcases {
		f.Add(tc) // Use f.Add to provide a seed corpus
	}
	f.Fuzz(func(t *testing.T, orig string) {
		rev := Reverse(orig)
		doubleRev := Reverse(rev)
		if orig != doubleRev { // 天才
			t.Errorf("Before: %q, After: %q", orig, doubleRev)
		}
		if utf8.ValidString(orig) && !utf8.ValidString(rev) {
			t.Errorf("Reverse produed invalid UTF-8 string %q", rev)
		}
	})
}
```
-> When fuzzing, you can’t predict the expected output, since you don’t have control over the inputs.

-> Note the syntax differences between the unit test and the fuzz test:

- The function begins with FuzzXxx instead of TestXxx, and takes `*testing.F` instead of `*testing.T`
- Where you would expect to see a `t.Run` execution, you instead see `f.Fuzz` which takes a fuzz target function whose parameters are `*testing.T` and the types to be fuzzed. The inputs from your unit test are provided as seed corpus inputs using `f.Add`.

Run `FuzzReverse` with fuzzing, to see if any randomly generated string inputs will cause a failure. This is executed using `go test` with a new flag, `-fuzz`, set to the parameter `Fuzz`.
```
$ go test -fuzz=Fuzz
```
-> 如果失败了，会生成一个testdata/fuzz/FuzzReverse的dir，里面装着会导致失败的input

-> 这是如果再次执行之前（没有flag的） go test 也会导致失败

-> 持续修改并测试

[link](https://developer.so-tech.co.jp/entry/2022/08/31/110108)
![スクリーンショット 2023-08-12 14 10 10](https://github.com/user-attachments/assets/7f9b775e-841b-44df-983c-18880b8eec41)


# [Writing Web Applications](https://go.dev/doc/articles/wiki/)

**Building a simple web application.**
```
// wiki.go
package main

import (
	"fmt"
	"os"
)

type Page struct {
	Title string
	Body []byte
}

func (p *Page) save() error {
	filename := p.Title + ".txt"
	return os.WriteFile(filename, p.Body, 0600)
}
```
-> This is a method named save that takes as its receiver p, a pointer to Page . It takes no parameters, and returns a value of type error

-> save没有参数input，只有一个指针？

-> 0600: indicates that the file should be created with read-write permissions for the current user only
```
// wiki.go
func loadPage(title string) (*Page, error) {
	filename := title + ".txt"
	body, err := os.ReadFile(filename)
	if err != nil {
		return nil, err
	}
	return &Page{Title: title, Body: body}, nil
}

func main() {
	p1 := &Page{Title: "TestPage", Body: []byte("This is a sample Page.")}
	p1.save()
	p2, _ := loadPage("TestPage")
	fmt.Println(string(p2.Body))
}
```
执行
```
go build wiki.go
```
-> 生成binary executable file
```
./wiki
```
-> 
```
This is a sample Page.
```
Full example of simple webserver:
```
package main

import (
    "fmt"
    "log"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}

use Handler: 

func viewHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/view/"):]
	p, _ := loadPage(title)
	fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}
```

-> The function then loads the page data, formats the page with a string of simple HTML, and writes it to `w`, the `http.ResponseWriter`. 
```
func main() {
    http.HandleFunc("/view/", viewHandler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```
create “test.txt”文件，写入“Hello World”

`go build wiki.go` & `./wiki`执行之后，access http://localhost:8080/view/test 。就会显示：
<img width="332" alt="スクリーンショット 2023-08-13 17 05 34" src="https://github.com/user-attachments/assets/a547aa9e-0519-4a8c-8569-cb6545c9d4f8">

*该教程剩下的部分一眼带过
