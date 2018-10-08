FROM golang:1.10-alpine3.7 AS build-env
COPY hello-world.go /tmp
RUN cd /tmp && go build -o goapp hello-world.go

# final stage
FROM scratch
WORKDIR /app
COPY --from=build-env /tmp/goapp /app/
ENTRYPOINT ["./goapp"]