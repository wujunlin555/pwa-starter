# PWA Starter 多阶段构建 Dockerfile
FROM node:18-alpine AS builder

# 设置工作目录
WORKDIR /app

# 安装 git（PWA Starter 可能需要）
RUN apk add --no-cache git

# 复制 package.json 和 package-lock.json
COPY package*.json ./

# 安装依赖
RUN npm ci --only=production

# 复制源代码
COPY . .

# 构建 PWA 应用
RUN npm run build

# 生产阶段 - 使用 Nginx 提供 PWA 支持
FROM nginx:alpine

# 安装额外工具（可选）
RUN apk add --no-cache curl

# 复制构建结果
COPY --from=builder /app/dist /usr/share/nginx/html

# 复制 PWA 专用的 Nginx 配置
COPY --from=builder /app/dist/manifest.json /usr/share/nginx/html/
COPY --from=builder /app/dist/sw.js /usr/share/nginx/html/

# 创建 PWA 友好的 Nginx 配置
RUN cat > /etc/nginx/conf.d/default.conf << 'EOF'
server {
    listen 80;
    server_name localhost;
    
    root /usr/share/nginx/html;
    index index.html;
    
    # PWA 必需的 HTTPS 头部（开发环境模拟）
    add_header 'Service-Worker-Allowed' '/';
    
    # 处理 SPA 路由
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # Service Worker 缓存策略
    location /sw.js {
        add_header Cache-Control "no-cache, no-store, must-revalidate";
        add_header Pragma "no-cache";
        add_header Expires "0";
    }
    
    # Manifest 文件
    location /manifest.json {
        add_header Content-Type "application/manifest+json";
        add_header Cache-Control "no-cache, no-store, must-revalidate";
    }
    
    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # PWA 图标缓存
    location ~* \.(webp|png)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # 安全头部
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
    add_header Referrer-Policy "strict-origin-when-cross-origin";
    
    # PWA 安装提示支持
    add_header 'Cross-Origin-Embedder-Policy' 'require-corp';
    add_header 'Cross-Origin-Opener-Policy' 'same-origin';
    
    # 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/javascript
        application/xml+rss
        application/json
        application/manifest+json;
}
EOF

# 健康检查
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD curl -f http://localhost/ || exit 1

# 创建非 root 用户（可选，增强安全性）
RUN addgroup -g 1001 -S pwagroup && \
    adduser -S pwauser -u 1001 -G pwagroup

# 暴露端口
EXPOSE 80

# 启动 Nginx
CMD ["nginx", "-g", "daemon off;"]
