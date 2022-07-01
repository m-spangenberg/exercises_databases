sudo docker exec -it post-dev bash
createdb dvdshop_db -U postgres
psql dvdshop_db -U postgres