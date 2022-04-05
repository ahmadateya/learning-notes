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
	

- [Should I commit vendor directory with go mod?](https://stackoverflow.com/questions/60865004/should-i-commit-vendor-directory-with-go-mod)
- `go install`
	- If there are replace or exclude directives in the module (you are tring to install), the correct installation method is to clone the source and install it.
	- resources
		- [go install github.com/dmacvicar/terraform-provider-libvirt@latest - shows error](https://stackoverflow.com/questions/69807151/go-install-github-com-dmacvicar-terraform-provider-libvirtlatest-shows-error)
		- [no replace directive allowed for "go install" ?](https://groups.google.com/g/golang-nuts/c/igwFOH-fWqI?pli=1)


- [panic: dial tcp 127.0.0.1:3306: connect: connection refused](https://stackoverflow.com/questions/57566060/panic-dial-tcp-127-0-0-13306-connect-connection-refused)
