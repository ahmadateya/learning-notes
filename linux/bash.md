# My notes from learning and using Linux bash


- **positional parameters** => params that come from calling the bash script
	- Parameters start from `$1` to `$9`
	- `$*` to get all the parameters
	- `$#` to get the number of all parameters 
- **Note:** there is a difference in the bash syntax depending on you are following bash syntax or POSIX syntax
- `$?` Captures the value of the last command
- When you create an `env` variable using `export` => it will not be persistent, it will be gone after the session ends
- **$PATH** is a very important variable because it's the variable that the system looks at if you typed any command, and it consists of all the available paths to the binaries folders in the system.
	- You can use it to append a path or add a new path to this list and 

- have a look on 
	- File conditionals
	- string operators


## Resources 
- [Bash cheat sheet](https://devhints.io/bash)
