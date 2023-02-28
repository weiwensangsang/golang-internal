# Packages

Go language uses **packages** to organize source code and implement namespace management. Any Go language program must belong to a **package**, that is, package <pkg_name> must be written at the beginning of each go program.

Go language **packages** generally meet the following three conditions:

1. All go files at the same level in the same directory should belong to a **package**;
2. The name of the **package** can be different from the name of the directory, but the same name is recommended;
3. A Go language program has one and only one main function, which is the entry function of the Go language program and must belong to the main **package**. If there is no or more than one, an error will be reported when the Go language program is compiled;





### Import

When organizing Go code structure and referencing packages, a concept called **GOPATH** will inevitably be involved, so what exactly is it?

**GOPATH** is an environment variable used by the GO language. It uses an absolute path to provide the working directory of the project. It is suitable for processing complex projects composed of a large number of Go language source codes and multiple packages. In actual use, you can first check the current GOPATH value through the command **go env**, and then decide whether it needs to be reset.



```
GOARCH="amd64"
GOBIN=""
GOCACHE="/Users/zhen/Library/Caches/go-build"
GOENV="/Users/zhen/Library/Application Support/go/env"
GOEXE=""
GOEXPERIMENT=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOINSECURE=""
GOMODCACHE="/Users/zhen/go/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="darwin"
GOPATH="/Users/zhen/go"
```



#### Import Format

There are four common import formats for Go packages. The following uses the fmt package as an example to illustrate:

1. The standard quotes import fmt

2. Set the alias reference import format_go fmt to omit the reference import . fmt. This kind of reference is equivalent to merging the namespace of the package fmt into the namespace of the current program, so it can be directly referenced without adding the prefix fmt.
3. Execute only the package initialization function import_fmt

In addition, when a Go program needs to import multiple packages, you can use single-line import or multi-line import. The two import forms are listed below:

single line import

```go
import "package1"
import "package2`
```

multiline import

```go
import (
     "package01"
     "package02"
)
```

In addition, the imported package must be used, otherwise an error will be reported when the program is compiled.



#### Go package reference initialization process

Assuming that the order of importing packages of a go language program is: main-> package A -> package B -> package C, and packages A, B, and C all implement the init() function, then the order of the entire initialization process is: C .init -> B.init -> A.init -> main -> package A -> package B -> package C.

#### Export of identifiers in packages

If a package wants to refer to another package's identifier (such as structure, variable, constant, function, etc.), it must be exported first. The specific method is to ensure that the first letter is capitalized (the first letter is lowercase) when defining these identifiers. identifiers can only be referenced within the package). In addition, in the exported structure or interface, only the fields and methods with **capital letters** can be accessed outside the package.