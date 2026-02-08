# 阶段一：构建
FROM golang:1.21-alpine AS builder

WORKDIR /app

# 安装依赖
RUN apk add --no-cache git make

# 设置 Go 国内代理
ENV GOPROXY=https://goproxy.cn,direct

# 复制 go.mod 和 go.sum
COPY go.mod go.sum ./
RUN go mod download

# 复制源码并构建
COPY . .
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -ldflags "-s -w" -o geekai-api main.go

# 阶段二：运行
FROM alpine:3.18

WORKDIR /var/www/app

# 安装时区数据
RUN apk add --no-cache tzdata ca-certificates

# 复制二进制文件
COPY --from=builder /app/geekai-api .

# 创建必要目录
RUN mkdir -p logs static data

EXPOSE 5678

CMD ["./geekai-api"]
