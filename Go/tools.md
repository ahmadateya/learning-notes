# Notes about the tooling that I am using in the Go development

- ### General
  - use `go fmt` to format your code
  
- ### Circular Dependency Detection Tools, [Good article](https://jogendra.dev/import-cycles-in-golang-and-how-to-deal-with-them)
	- [godepgraph](https://github.com/kisielk/godepgraph) a dependency graph visualization tool.
		- It display graph in [Graphviz](http://graphviz.org/) dot format, install it from [here](https://graphviz.org/download/)
