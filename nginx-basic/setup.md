## Frontend Configuration

### Add Dockerfile
```bash
# Stage 1: Build
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Serve with Nginx
FROM nginx:alpine

COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Add base in vite.config.js
```bash
export default defineConfig({
  base: '/app2/',
  // other Vite config options...
});
```

## nginx.conf file
```bash
events {}

http {
    server {
        listen 80;

        location /app1/ {
            proxy_pass http://template-demo:80/;
        }

        location /app2/ {
            proxy_pass http://poc:80/;
        }
    }
}
```

## docker-compose.yml file

```bash
services:
  template-demo:
    build: ./Templet-demo
    container_name: template-demo
    restart: always

  poc:
    build: ./templet-POC
    container_name: poc
    restart: always

  reverse-proxy:
    image: nginx:alpine
    container_name: reverse-proxy
    ports:
      - "4000:80"
    volumes:
      - ./reverse-proxy/nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - template-demo
      - poc
    restart: always
```
## Running Application
```bash
docker compose up -d --build
```

Access the apps via the reverse proxy 
1. http://localhost:4000/app1
2. http://localhost:4000/app2
