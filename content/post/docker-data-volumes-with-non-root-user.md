+++
date = "2014-11-11T17:22:51+08:00"
title = "Docker data volumes with non-root user"

+++

One way of using docker volumes with non-root user, with camlistore as an example.
camlistored is built beforehand with ```go run make.go -docker_camlistored```.

Dockerfile:
```
FROM busybox

# create user for camlistore data storage
# use 3179 as uid to avoid collision
RUN adduser -u 3179 -D camli
USER camli
COPY camlistored /home/camli/camlistored
WORKDIR /home/camli
RUN mkdir -p .config/camlistore \
        && mkdir -p var/camlistore
VOLUME ["/home/camli/.config/camlistore", "/home/camli/var/camlistore"]
EXPOSE 3179
CMD ["./camlistored"]
```

Build the images and run the containers: (e.g. user/camlistore)

```
sudo docker create --name camlistore-data user/camlistore true
sudo docker run -d --name camlistore -p 3179:3179 --volumes-from camlistore-data user/camlistore
```

Keypoints:

1. Create a specific user (e.g. camli)
2. Create directories for the volumes with that user before running VOLUME (i.e. ```USER camli```)
3. Create the data container
4. Run the server container mounting the volumes from the data container with --volumes-from
