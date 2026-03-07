```bash
project
│
├── docker-compose.yml
│
├── reverse-proxy
│   └── nginx.conf
│
├── frontend
│   ├── Dockerfile
│   ├── nginx.conf
│   └── React source
│
├── gateway
│
├── eureka
│
└── services
    ├── app-1
    └── app-2
```

# Architecture
```bash
Browser
   │
   ▼
Nginx (port 4000)
   │
   ├── /            → React frontend
   │
   └── /api/*       → Spring API Gateway
                         │
                         └── /api/orders/** → app-2
```