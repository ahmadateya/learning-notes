# My notes from learning filesystem in Linux

## File System Overview & File Structure
- In linux Everything is a file (even commands, pictures, directories, devices (mouse, keyboard ...etc))
- Linux file system structure
    - <img src="https://github.com/ahmadateya/learning-notes/blob/main/assets/images/linux-file-system-structure-final-4.png" width="600" height="400">

- **/usr/local**
    - <img src="https://github.com/ahmadateya/learning-notes/blob/main/assets/images/Screenshot%20from%202021-09-14%2014-39-29.png" width="500" height="200">

- **/usr/local** vs. **/opt**
    - <img src="https://github.com/ahmadateya/learning-notes/blob/main/assets/images/Screenshot%20from%202021-09-14%2014-41-33.png" width="500" height="200">

- **File Links**
    - <img src="https://github.com/ahmadateya/learning-notes/blob/main/assets/images/hardlink-vs-softlink.jpg" width="400" height="200">
    - [Hard links vs. soft links (symbolic links)](https://stackoverflow.com/a/29786294/10179972)
    - Some Notes about Links
      1. you can only hardlink files, not directories; but can softlink directories 
      2. only hardlink to files on the same filesystem; but can softlink to files on different filesystems
    - **Commands for Files hard link & soft link (symlinks)**
      - **ln -s** for soft link (symlink)
      - **ln** for hard link
		
- **File Permissions**
    - any file of the system has an owner that the only one who can change the file permissions except root
    - <img src="https://github.com/ahmadateya/learning-notes/blob/main/assets/images/Screenshot%20from%202022-05-03%2005-40-01.png" width="400" height="200">
    - [Sticky Bit, SUID and SGID](https://tecadmin.net/understanding-sticky-bit-suid-and-sgid-in-linux/)
    - **Commands for file permissions**
      - **chmod** to change the file permissions
      - **chown [OPTIONS] USER[:GROUP] FILE(s)** to change the owner of the file, but only the root can do this
      - **chgrp group_name path/to/file** to change the group of the file
      - **stat -c "%a %n" path/to/file** to see the permissions of a file in octal mode, [reference](https://askubuntu.com/questions/152001/how-can-i-get-octal-file-permissions-from-command-line)
      
- **Search for files**
    - **Commands for search for files**
      - **find [path/to/directory] [search_parameters]** to search for files
        - No path => search in the current directory
        - **-name** to search for a specific file name
          - **-iname** to search for a case-insensitive file name
          - **-name "f*"** wildcard search
        - **-mmin [minute]** to search for files in the last [minute] minutes
        - **mtime [days]** to search for files in the last [days]
        - **-type** to search for files with the type of file
        - **-perm [permission]** to search for files with the permission of file
          - <img src="https://github.com/ahmadateya/learning-notes/blob/main/assets/images/Screenshot%20from%202022-05-03%2006-35-48.png" width="400" height="200">
        - 