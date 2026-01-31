```Dockerfile
FROM golang:1.24.1 AS builder

WORKDIR /main

COPY .env.secret .

RUN go env -w GOPRIVATE=github.com/comvex-jp
RUN token=$(grep 'GITHUB_ACCESS_TOKEN=' .env.secret | sed 's/^.*=//') && \
    printf "[url \"https://%s:x-oauth-basic@github.com/\"]\n  insteadOf = https://github.com/\n" "$token" >> ~/.gitconfig

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN go build -o source .

FROM gcr.io/distroless/static-debian12 AS runner
WORKDIR /root/
COPY --from=builder /main/source ./
COPY --from=builder /main/infra/database/migrations ./database/migrations


EXPOSE 50051


CMD ["./source", "serve"]
```