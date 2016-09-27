## Docker Study

docker run -d -p 80:80 --name webserver nginx

docker ps

docker start webserver

docker stop webserver

docker rm -rf webserver

docker rmi <imageId> | <imageName>

docker rmi nginx