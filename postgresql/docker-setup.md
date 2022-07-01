## Quick Setup: PostgreSQL in Docker

I'm going to set up a container environment for PostgreSQL and pgAdmin.

Remember to replace `YOURPASSWORD` and `YOUREMAIL`!

```bash
# make a data directory
mkdir data
 
# pull the postgres and admin panel images
sudo docker pull postgres
sudo docker pull dpage/pgadmin4
 
# create a docker container for postgresql and the admin panel
sudo docker run --name post-dev -e 'POSTGRES_PASSWORD=YOURPASSWORD' -v ${HOME}/path/to/folder/data:/var/lib/postgresql/data -p 5551:5551 -d postgres
    
sudo docker run --name post-admin -p 80:80 -e 'PGADMIN_DEFAULT_EMAIL=YOUREMAIL' -e 'PGADMIN_DEFAULT_PASSWORD=YOURPASSWORD' -d dpage/pgadmin4
 
# you can enter the postgresql container
sudo docker exec -it post-dev bash

# and get a shell for postgres
psql -h localhost -U postgres

# See: https://www.postgresql.org/docs/current/tutorial.html 

# or connect to it with the admin panel, just grab the postgresql containers IP first.
sudo docker inspect post-dev {{ .NetworkSettings.Networks.$network.IPAddress }}
# It should be something like: 172.17.0.2
# go to 127.0.0.1:80 to connect to the admin panel and add the server
```