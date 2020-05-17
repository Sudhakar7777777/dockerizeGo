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
