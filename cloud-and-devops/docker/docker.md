# Notes and other things related to Docker


* To export or backup a DB running in docker container to a file you can use these commands:
	*  
	* PostgreSQL `docker exec -it <container-name> pg_dump --username=<user> --dbname=<dbname> --port=<port> --column-inserts > path/to/dump_`date +%d-%m-%Y"_"%H_%M_%S`.sql`
		* you can use `pg_dump` or `pg_dumpall` 
	* MySQL `docker exec CONTAINER /usr/bin/mysqldump -u <user> --password <password> <name_db> > path/to/file.sql`

* To import a db from file to a DB running in docker container you can use these commands:
	* PostgreSQL `docker exec -i <container-name> psql --username=<user> --dbname=<dbname> --port=<port> < path/to/file.sql`
	* MySQL `docker exec -i <container-name> mysql -u <user> -p <password> <name_db> < path/to/file.sql`
* To get the container IP exposed to the host machine use:
	* `docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container-name>`





## Resources for backing up the DB
* [Backup/Restore a dockerized PostgreSQL database](https://stackoverflow.com/questions/24718706/backup-restore-a-dockerized-postgresql-database).
* [How to copy a file from remote server to local machine? [closed]](https://stackoverflow.com/questions/28869004/how-to-copy-a-file-from-remote-server-to-local-machine).


## Resourses for Docker Compose Files
* [Awesome Docker compose](https://github.com/docker/awesome-compose/tree/master/nginx-golang-postgres).
