# Common Issues I found while development with Go


- Forking a package introducing problems in the `go.mod` file and including this package in another project/package
	- [How to fix parsing go.mod module declares its path as "x" but was required as "y"](https://stackoverflow.com/questions/61311436/how-to-fix-parsing-go-mod-module-declares-its-path-as-x-but-was-required-as-y)
	- [Solving module declares its path as X but was required as Y](https://markcz.wordpress.com/2020/04/14/solving-module-declares-its-path-as-x-but-was-required-as-y/) this solution worked for me


- Including private packages in `go.mod`, especially when using Docker
	- [go get for private repos in docker](https://divan.dev/posts/go_get_private/)
	- [How to use Go Modules with Private Git repository](https://dev.to/gopher/how-to-use-go-modules-with-private-git-repository-53b4) the tutorial that I followed
	- [Go Modules with Private GIT Repository](https://medium.com/swlh/go-modules-with-private-git-repository-3940b6835727)

