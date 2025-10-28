
# Docker Installation and SonarQube Setup Guide
Sonarqube setup in ubuntu. 

## 1. Update System
```bash
sudo apt update -y
sudo apt upgrade -y
```

## 2. Installation
```bash
# Install Required Packages
sudo apt install -y ca-certificates curl gnupg lsb-release

# Add Docker’s Official GPG Key & Repo
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker Engine
sudo apt update -y
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## 3. Start and Enable Docker
```bash
sudo systemctl start docker
sudo systemctl enable docker

# Allow Docker for Non-root User
sudo usermod -aG docker $USER
newgrp docker
```

## 4. Increase Virtual Memory Mapping
```bash
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
```

## 5. Sonarqube Installation
```bash
mkdir sonarqube
cd sonarqube

# create docker compose
nano docker-compose.yml

# put this in docker compose
version: "3.8"
services:
  sonarqube:
    image: sonarqube:latest
    container_name: sonarqube
    restart: unless-stopped
    ports:
      - "9000:9000"
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar
    volumes:
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs
    depends_on:
      - db

  db:
    image: postgres:15
    container_name: postgres
    restart: unless-stopped
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
      - POSTGRES_DB=sonar
    volumes:
      - postgresql:/var/lib/postgresql

volumes:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
```

## 6. To run sonarqube
```bash
docker compose up -d

# access it at http://<your-server-ip>:9000
``` 

