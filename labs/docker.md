# Build and run Laravel with Docker

## Build images

#### Using Docker Compose
1. Set environment variables in your shell for images name or edit your images name in `docker-compose.yml`.
2. Build images with docker compose.
    ```
    docker-compose build 
    ```
#### Using Dockerfile
Build images from a Dockerfile in `docker` directory.
```
docker build -t YOUR_IMAGE_NAME:YOUR_IMAGE_TAG PATH/TO/Dockerfile
```

> The following command can use to remove intermediate images produced when using multi stage build in Dockerfile:
> ```
> docker rmi -f $(docker images --quiet --filter "dangling=true")
> ```


## Run images

#### Local development with Phpmyadmin and Mysql
```
docker-compose -f docker-compose.yml -f docker-compose.local.yml up
```

#### Run development or production environment locally
```
// Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

// Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```