Here, I'll use one Nginx reverse proxy and this will route traffic

 ## Artchitecture
 ```bash  
                Internet
                    ↓
        Reverse Proxy (Nginx container)
                    ↓
        ┌───────────────┬───────────────┐
        ↓               ↓
      app-1           app-2           app-3
```

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
│   ├── Dockerfile
│   └── (react project)
│
└── frontend2/
    ├── Dockerfile
    └── (react project)
```