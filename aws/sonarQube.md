# SonarQube Installation with Java 21 (Amazon Corretto) on AWS Linux

This guide walks you through the steps of installing Java 21 and SonarQube Community Edition on a Linux system, including necessary permissions and cleanup procedures.

## 1. Install Java 21 (Amazon Corretto)
Java installation for sonarqube

```bash
# Install Amazon Corretto 21 (Headless)
sudo yum install java-21-amazon-corretto-headless

# Install Java 21 development tools
sudo yum install -y java-21-amazon-corretto-devel

# Verify Java installation
java -version

# Verify Java compiler installation
javac -version

```
## 1. Install SonarQube
```bash
## Navigate to the /opt directory
cd /opt

# Download SonarQube Community Edition
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-25.9.0.112764.zip

# Unzip the downloaded SonarQube package
sudo unzip sonarqube-25.9.0.112764.zip

# Create a system user for SonarQube
sudo useradd -r -s /bin/bash sonarqube
sudo chown -R sonarqube:sonarqube /opt/sonarqube-25.9.0.112764/
sudo chmod -R 755 /opt/sonarqube-25.9.0.112764/

# Switch to the sonarqube user and start SonarQube
sudo -u sonarqube /opt/sonarqube-25.9.0.112764/bin/linux-x86-64/sonar.sh start

# Check SonarQube status
sudo -u sonarqube /opt/sonarqube-25.9.0.112764/bin/linux-x86-64/sonar.sh status

# Sonarqube runnning on
http://<your-server-ip>:9000
```

## Fix directory permission in case of failure 
```bash 
# Remove temporary files in SonarQube's temp directory
sudo -u sonarqube rm -rf /opt/sonarqube-25.9.0.112764/temp/*
sudo -u sonarqube rm -rf /opt/sonarqube-25.9.0.112764/logs/*

# Ensure the data directory has the correct ownership
sudo chown -R sonarqube:sonarqube /opt/sonarqube-25.9.0.112764/data/
```



