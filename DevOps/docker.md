# Notes and other things related to Docker


* To export a DB running in docker container to a file you can use these commands:
	- PostgreSQL `docker exec -i <container-name> pg_dump --username=<user> --dbname=<dbname> --port=<port> --column-inserts > path/to/file.sql`
	- MySQL `docker exec CONTAINER /usr/bin/mysqldump -u <user> --password <password> <name_db> > path/to/file.sql`

* To import a db from file to a DB running in docker container you can use these commands:
	- PostgreSQL `docker exec -i <container-name> psql --username=<user> --dbname=<dbname> --port=<port> < path/to/file.sql`
	- MySQL `docker exec -i <container-name> mysql -u <user> -p <password> <name_db> < path/to/file.sql`
* To get the container IP exposed to the host machine use:
	- `docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container-name>`
