## Frontend Configuration

## Folder Structure
```bash
project/
│
├── docker-compose.yml
│
├── reverse-proxy/
│   └── nginx.conf
│
├── frontend1/
│
└── frontend2/
```

### Add base 

```bash
# For react add base in vite.config.js
export default defineConfig({
  base: '/app2/',
  // other Vite config options...
});

# Angular - base in index.html
<base href="/app2/">
```

## nginx.conf file
```bash
events {}

http {
    server {
        listen 80;
        root /usr/share/nginx/html;

        location /app1/ {
            try_files $uri $uri/ /app1/index.html;
        }

        location /app2/ {
            try_files $uri $uri/ /app2/index.html;
        }
    }
}
```

## docker-compose.yml file
```bash
# create dist
ng build --configuration production --base-href /app2/
```

```bash
services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "4000:80"
    volumes:
      - ./reverse-proxy/nginx.conf:/etc/nginx/nginx.conf
       # Mount Angular build output
      - ./app-1/dist/app1/browser:/usr/share/nginx/html/app1:ro
      - ./app-2/dist/app-2/browser:/usr/share/nginx/html/app2:ro

```
## Running Application
```bash
docker compose up -d --build
```

Access the apps via the reverse proxy 
1. http://localhost:4000/app1/
2. http://localhost:4000/app2/
