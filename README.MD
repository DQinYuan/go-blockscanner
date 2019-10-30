# go-blockscanner

a library to visit all basic blocks of go source code.(Borrow a lot of code from `go test`)


an example


```go
package main

import (
	"fmt"
	"github.com/dqinyuan/go-blockscanner"
	"go/ast"
	"go/parser"
	"go/token"
	"log"
)

func main() {
	code := `package test
func printStu(){
    a := 10
	if a > 10 {
        a++
    }	
}
`

	codeBytes := []byte(code)
	fileSet, afile, err := parse(codeBytes)
	if err != nil {
		log.Fatalf("parse error: %v\n", err)
	}

	scanner := blockscanner.NewBlockScanner(fileSet, codeBytes, func(startLine, endLine token.Position, blockNum int) (insert string) {
		fmt.Printf("block: %d, start: %d, end: %d\n", blockNum, startLine.Line, endLine.Line)
		return "//aaa"
	})

	ast.Walk(scanner, afile)
	/*
	block: 0, start: 2, end: 4
    block: 1, start: 4, end: 6
	*/

	// content after insert
	fmt.Println(string(scanner.Res()))
	/*
	package test
    func printStu(){//aaa
        a := 10
	    if a > 10 {//aaa
            a++
        }	
    }
	*/
}


func parse(content []byte) (*token.FileSet, *ast.File, error) {
	fset := token.NewFileSet()
	aFile, err := parser.ParseFile(fset, "", content, parser.ParseComments)
	if err != nil {
		return nil, nil, err
	}

	return fset, aFile, nil
}

```