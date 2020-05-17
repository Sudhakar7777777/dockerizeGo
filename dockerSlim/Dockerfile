### Build executable
## Dockerfile References: https://docs.docker.com/engine/reference/builder/

# Start from the latest golang base image
FROM golang:latest AS buildImage

# Add maintainer info
LABEL maintainer="Sudhakar (https://github.com/Sudhakar7777777)"

# Create app folder and add files
RUN mkdir /app
ADD . /app/

# Set the Current Working Directory inside the container
WORKDIR /app

# Set environment configs
ENV GOOS linux
ENV CGO_ENABLED 0

# Build the Go app
RUN go build -v -a -installsuffix cgo -o main .


### Build new base for deploying standalong executable
FROM scratch

# Copy certificate file used for supporting https connections
COPY --from=buildImage /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/

# Copy the pre-built binary file from the previous stage
COPY --from=buildImage /app/main /

# Command to run the executable
CMD ["/main"]
