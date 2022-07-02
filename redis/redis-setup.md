## Quick Setup: Redis in Docker

```bash
# pull the postgres and admin panel images
sudo docker pull redis

# create a docker container for Redis
sudo docker run --name redis-dev -v ${HOME}/path/to/data:/var/lib/redis/data -p 6379:6379 -d redis redis-server --save 60 1 --loglevel warning --requirepass YOURPASSWORD

# you can enter the postgresql container
sudo docker exec -it redis-dev bash

# interact with redis through its cli
redis-cli
```
