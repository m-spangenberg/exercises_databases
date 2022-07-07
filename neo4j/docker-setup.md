## Quick Setup: Neo4j in Docker

I'm going to set up a container environment for Neo4j.

Remember to replace `YOURPASSWORD`.

```bash
# make a data directory
mkdir data
 
# pull the neo4j and admin panel images
sudo docker pull neo4j
 
# create a docker container for neo4j and the admin panel
sudo docker run --name neo-dev -e "NEO4J_AUTH=neo4j/YOURPASSWORD" -v ${HOME}/path/to/folder/data:/data -p 7474:7474 -p 7687:7687 -d neo4j
 
# Default is neo4j/neo4j
# go to http://localhost:7474 to access neo4j

# More info: 
# https://neo4j.com/developer/docker/
# https://neo4j.com/developer/docker-run-neo4j/
```