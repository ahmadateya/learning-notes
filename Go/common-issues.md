# Common Issues I found while development with Go

- The problems that I was facing is forking a Go pacakge to add some features and include it in my private repo inside my organization, I searched to see of there is a tutorial or something I could follow to solve the issues I found, and these are the results of my search:
	- Forking a package introducing problems in the `go.mod` file and including this package in another project/package
		- [How to fix parsing go.mod module declares its path as "x" but was required as "y"](https://stackoverflow.com/questions/61311436/how-to-fix-parsing-go-mod-module-declares-its-path-as-x-but-was-required-as-y)
		- [Solving module declares its path as X but was required as Y](https://markcz.wordpress.com/2020/04/14/solving-module-declares-its-path-as-x-but-was-required-as-y/) this solution worked for me
			- **Note:** You don't have to use `replace` if you tagged your forked repo (I think this is nicer than using `replace` I think its confusing)
		- [Using forked package import in Go](https://stackoverflow.com/questions/14323872/using-forked-package-import-in-go)

	- Including private packages in `go.mod`, especially when using Docker
		- [go get for private repos in docker](https://divan.dev/posts/go_get_private/)
		- [How to use Go Modules with Private Git repository](https://dev.to/gopher/how-to-use-go-modules-with-private-git-repository-53b4) the tutorial that I followed
			- ### Remember: don't use any `go get` commands in `dockerfile` before setting the git configuration 
		- [Go Modules with Private GIT Repository](https://medium.com/swlh/go-modules-with-private-git-repository-3940b6835727)
	
