## Quick Setup: SQLite3 in Docker

```bash
# pull the Python images, it has native support for sqlite
sudo docker pull python:latest

# create a docker container for Python
sudo docker run --name python-dev -v ${HOME}/path/to/data:/ -d python

# you can enter the Python container
sudo docker exec -it python-dev bash

# create a sqlite database
...
```
