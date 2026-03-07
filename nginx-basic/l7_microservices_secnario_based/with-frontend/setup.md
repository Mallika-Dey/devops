# Frontend docker file
```bash
# ---------- Build Stage ----------
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# ---------- Production Stage ----------
FROM nginx:alpine

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 4000

CMD ["nginx", "-g", "daemon off;"]
```

# Frontend nginx.conf
```bash
server {
    listen 80;

    server_name localhost;

    root /usr/share/nginx/html;
    index index.html;

    # React SPA routing
    location / {
        try_files $uri /index.html;
    }

    # Static asset caching
    location ~* \.(js|css|png|jpg|jpeg|gif|svg|ico)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }
}
```

# Reverse proxy nginx.conf
```bash
events {}

http {

log_format upstreamlog '$remote_addr - $host '
                       'upstream: $upstream_addr '
                       'status: $status';

access_log /var/log/nginx/access.log upstreamlog;

    upstream gateway_cluster {
        server api-gateway:8080;
    }

    upstream frontend_cluster {
        server frontend:80;
    }

    server {
        listen 80;

        # React frontend
        location / {
            proxy_pass http://frontend_cluster;
            proxy_set_header Host $host;
        }

        # API Gateway
        location /api/ {
            proxy_pass http://gateway_cluster;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

# Docker compose
```bash
services:

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "4000:80"
    depends_on:
      - api-gateway
      - frontend
    volumes:
      - ./reverse-proxy/nginx.conf:/etc/nginx/nginx.conf
    networks:
      - micro-net
      - front-net


  frontend:
    image: frontend:0.1
    container_name: react-frontend
    networks:
      - front-net


  eureka-server:
    image: eureka:1.1
    container_name: eureka-server
    ports:
      - "8761:8761"
    networks:
      - micro-net


  api-gateway:
    image: gateway:1.1
    container_name: api-gateway
    depends_on:
      - eureka-server
    networks:
      - micro-net


  app-1-instance-1:
    image: app-1:1.1
    container_name: app-1-instance-1
    networks:
      - micro-net


  app-1-instance-2:
    image: app-1:1.1
    container_name: app-1-instance-2
    networks:
      - micro-net


  app-2-instance-1:
    image: app-2:1.1
    container_name: app-2-instance-1
    networks:
      - micro-net


networks:
  micro-net:
  front-net:
```

