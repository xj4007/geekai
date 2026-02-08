# 阶段一：构建
FROM node:18-alpine AS builder

WORKDIR /app

# 复制 package.json
COPY package*.json ./
RUN npm install

# 复制源码并构建
COPY . .
RUN npm run build

# 阶段二：Nginx 运行
FROM nginx:alpine

# 复制构建产物
COPY --from=builder /app/dist /usr/share/nginx/html

# 复制 nginx 配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 8080

CMD ["nginx", "-g", "daemon off;"]
