FROM golang:latest

LABEL maintainer="Gary Yang <ygb_1023@sina.com>"

RUN mkdir /app
ADD hello-world.go /app/
WORKDIR /app
RUN go build -o hello-world hello-world.go

EXPOSE 8081
CMD ["./hello-world"]
