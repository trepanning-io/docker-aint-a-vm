FROM golang:1.10-alpine3.7 AS build-env
ADD hello-world.go /tmp
RUN cd /tmp && go build -ldflags="-s -w" -o goapp hello-world.go

# final stage
FROM scratch
WORKDIR /app
COPY --from=build-env /tmp/goapp /app/
ENTRYPOINT ["./goapp"]