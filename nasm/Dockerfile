FROM alpine:3.7 AS build-env
WORKDIR /src
RUN apk update && apk add --no-cache nasm build-base && \
    rm -rf /var/cache/apk/*

COPY hello.asm /src
RUN nasm -f elf hello.asm
RUN ld -m elf_i386 -s -o hello hello.o

# final stage
FROM scratch
WORKDIR /app
COPY --from=build-env /src/hello /app/hello
ENTRYPOINT ["./hello"]