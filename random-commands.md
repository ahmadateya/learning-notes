# Random Commands that I regularly use and regularly forget 😅

* Upload and download files over SSH by (Secure copy protocol)
	* Download `scp <username>@<hostname>:<path/on/remote/host> </path/to/local/dir>`
	* Upload `scp </path/to/local/dir> <username>@<hostname>:<path/on/remote/host>`

* Delete .swp files
	* `find . -name 'filename.swp' -print0 | xargs -0 rm -i --`
	* `rm .oetest.tech.swp`
 

