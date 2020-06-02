# Docker cheat sheet

Build image by Dockerfile

```sh
docker build -t image-name .
```

Run container

```sh
docker run -d 8080:8080 --name container-name image-name:latest
```

Get logs

```
docker logs -f container-name
```

Get shell

```sh
docker exec -it container-name sh
```

Fast shortcuts

```sh
gradle clean build && docker build -t image-name . && docker run -d -p 8080:8080 --name container-name image-name:latest
```

```sh
docker stop will-mock && docker rm will-mock 
```

