# Jenkins and Maven Setup on Amazon Linux 2023

## 1. Install Apache Maven

First, let's install Apache Maven.

```bash
cd /opt

# Download Maven 3.9.11 binary tar.gz
wget https://dlcdn.apache.org/maven/maven-3/3.9.11/binaries/apache-maven-3.9.11-bin.tar.gz

# Extract the tarball
tar xvf apache-maven-3.9.11-bin.tar.gz
```
## 2. Install Jenkins
### Add Jenkins Repository
```bash
cd /etc/yum.repos.d/

# Download the Jenkins repo file
wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo

# Import the GPG Key
 rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key 
 ```
### Add EPEL Repository
```bash
# run this command
sudo tee /etc/yum.repos.d/epel.repo > /dev/null <<'EOF'
[epel]
name=Extra Packages for Enterprise Linux 9 - $basearch
baseurl=https://dl.fedoraproject.org/pub/epel/9/Everything/$basearch/
enabled=1
gpgcheck=1
gpgkey=https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-9
EOF
 ```
### Clean and Update Yum
```bash
# Clean all cached data
sudo dnf clean all

# Rebuild the yum cache
sudo dnf makecache
```
### install jenkins
```bash
# Install Jenkins
sudo yum install jenkins
# start jenkins
service jenkins start
service jenkins status

# configure jenkins on EC2:
get the pass from
cat /var/lib/jenkins/secrets/initialAdminPassword
```





