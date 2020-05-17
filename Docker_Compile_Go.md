# How to Compile GO application using Docker

## Manual Methods:

### Manual method: [Linux][ directly compile from github ]

This will work on Linux devices. If you are using mac this will not work

```
docker run --rm \
-v $(pwd):/go/bin \
golang go get github.com/Sudhakar7777777/goHello/...

./hello
```

See the compile file type:

```
goHello: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, not stripped
```

### Manual method: [Mac][ directly compile from github ]

Compiling Go app on Mac devices require additional setup.

```
export GOOS=darwin GOARCH=amd64

docker run --rm \
-v $(pwd):/go/bin/${GOOS}_${GOARCH} \
-e GOOS -e GOARCH \
golang go get github.com/Sudhakar7777777/goHello/...

./hello
```

See the compile file type:

```
goHello: Mach-O 64-bit executable x86_64
```

Note: The above will not work when go code use net or cgo packages.

### Manual method: [Mac][ clone and compile on local ]

```
export GOOS=darwin GOARCH=amd64

mkdir compileGo
cd compileGo

git clone git://github.com/Sudhakar7777777/goHello
cat goHello/hello.go

docker run --rm \
-v $(pwd)/goHello:/go/bin \
-v $(pwd)/goHello:/go/bin/${GOOS}_${GOARCH} \
-e GOOS -e GOARCH \
golang go get github.com/Sudhakar7777777/goHello/...

./goHello/hello
```

Use code from current folder

```
export GOOS=darwin GOARCH=amd64

mkdir compileGo
cd compileGo
ls -l pageSize.go

docker run --rm \
-v $(pwd):/go/src/pageSize \
-v $(pwd):/go/bin/${GOOS}_${GOARCH} \
-e GOOS -e GOARCH \
golang go install pageSize

./pageSize
```

## Dockerization

### Simple Dockerfile (see folder: dockerBasic)

This generates a container with huge size.

```
FROM golang:latest
RUN mkdir /app
ADD . /app/
WORKDIR /app
RUN go build -o main .
CMD ["/app/main"]
```

```
docker build -t page-size .
```

```
docker run page-size
```

### Slim Dockerfile (see folder: dockerSlim)

Use scratch as the base image for standlone executable

```
### Build executable

FROM golang:latest AS buildImage

RUN mkdir /app
ADD . /app/

WORKDIR /app

ENV GOOS linux
ENV CGO_ENABLED 0

RUN go build -v -a -installsuffix cgo -o main .

RUN find /go

### Build image to deploy, the individual executable

FROM scratch

COPY --from=buildImage /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=buildImage /app/main /

CMD ["/main"]
```

```
docker build -t -rm page-slim .

docker run page-slim
```

### Slim Plus Dockerfile (see folder: dockerSlimPlus)

Use alpine linux as the base for deployed app.

```
### Build executable

FROM golang:latest AS buildImage

RUN mkdir /app
ADD . /app/

WORKDIR /app

ENV GOOS linux
ENV CGO_ENABLED 0

RUN go build -v -a -installsuffix cgo -o main .

RUN find /go

### Build image to deploy, the individual executable

FROM alpine:latest

COPY --from=buildImage /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=buildImage /app/main /
COPY --from=buildImage /bin/bash /bin/bash

CMD ["/main"]
```

```
docker build -t -rm page-slim-plus .

docker run page-slim-plus
```

## sample docker commands

Remove intermediate images

```
docker rmi \$(docker images -f dangling=true -q )
```

### additional notes on edge cases

To compile go apps with net, use netgo like this example...

```
docker run -v \$(pwd):/go/bin --rm golang go get -tags netgo -installsuffix netgo github.com/golang/example/hello/...
```

-tags netgo instructs the toolchain to use netgo.

-installsuffix netgo will make sure that the resulting libraries (if any) are placed in a different, non-default directory. This will avoid conflicts between code built with and without netgo, if you do multiple go get (or go build) invocations.

### Handle certificate issues...

multiple options available. easy is to use Alpine image, where it has latest root certificates. Use like shown below...

```
FROM alpine:3.4
RUN apk add --no-cache ca-certificates apache2-utils
```

### note its, good idea to clean cache after installs... like...

> apt-get update && apt-get install ... && rm -rf /var/cache/apt/\*

so... for above 2 steps... you can use alphine + certification + netgo to tackle these issues...

## Reference:

1. [docker-golang](https://www.docker.com/blog/docker-golang/)
