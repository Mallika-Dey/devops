# Microservices Configuration
app-1, app-2 and gateway run on port 8080.
```bash
# eureka configuration

eureka:
  client:
    service-url:
      defaultZone: http://eureka-server:8761/eureka/ # must match the name with compose
    register-with-eureka: true
    fetch-registry: true
```

# nginx.conf
```bash
events {}

http {
log_format upstreamlog '$remote_addr - $host '
                           'upstream: $upstream_addr '
                           'status: $status';

                            access_log /var/log/nginx/access.log upstreamlog;


    upstream gateway_cluster {
        server api-gateway:8080;
        # server api-gateway-2:8080;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://gateway_cluster;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

# docker compose

```bash
services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "4000:80"
    depends_on:
      - api-gateway
    volumes:
      - ./reverse-proxy/nginx.conf:/etc/nginx/nginx.conf
    networks:
      - micro-net

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
    ports:
      - "8080:8080"
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

```