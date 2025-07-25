# 最简单的 PWA Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html

# 基本的 PWA 支持配置
RUN sed -i '/location \/ {/a\        add_header "Service-Worker-Allowed" "/";' /etc/nginx/conf.d/default.conf && \
    sed -i '/try_files/c\        try_files $uri $uri/ /index.html;' /etc/nginx/conf.d/default.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
