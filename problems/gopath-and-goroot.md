# GOPATH and GOROOT

Unlike other languages, there is no project statement in go, only packages, in which there are two important paths, GOROOT and GOPATH

The environment variables related to Go development are as follows:

- GOROOT: GOROOT is the installation directory of Go, (similar to java's JDK)
- GOPATH: GOPATH is our workspace, saving go project code and third-party dependent packages

Multiple GOPATHs can be set. Among them, the first one will be the default package directory. The package downloaded using go get will be in the src directory in the first path. When using go install, in which GOPATH is the package found? , the executable file will be generated in the bin directory under which GOPATH