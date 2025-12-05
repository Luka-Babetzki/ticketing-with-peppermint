# Ticketing with Peppermint

## Overview
Self-hosted ticketing system deployed via Docker on Ubuntu 22.04 LTS for simulating helpdesk workflows.

## Objectives
- Deploy Peppermint ticketing system inside a Docker container on Ubuntu
- Understand ticket assignment, prioritisation, and management workflows
- Learn to triage tickets based on impact and urgency

## Prerequisites
This project requires a baseline VMware environment. Complete the following from the [Home Lab Foundation](https://github.com/Luka-Babetzki/homelab-foundation) repository:

- VMware Workstation Pro installation
- Download required OS ISOs: Ubuntu 22.04
- Configure NAT network (VMnet8)

If you haven't set up your foundation environment yet, follow the complete guide [here](https://github.com/Luka-Babetzki/homelab-foundation).

## Technologies Used
- Docker Engine
- Docker Compose
- Peppermint (open-source ticketing platform)

## Architecture Design

![Network Topology Diagram](path/to/diagram.png)

## Implementation Steps

### 1. Install Docker on Ubuntu

Followed official Docker documentation to install Docker Engine and Docker Compose plugin.

**Add Docker's official GPG key and repository:**
```bash
# Update package index and install prerequisites
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings

# Download Docker's GPG key
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add Docker repository to apt sources
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

**Install Docker packages:**
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Verify installation:**
```bash
sudo docker run hello-world
```

**Challenge encountered:** Initial installation failed due to conflicting Docker packages from Ubuntu's default repositories. Resolved by removing all unofficial Docker packages before installing from official Docker repository (see Troubleshooting section).

### 2. Deploy Peppermint via Docker Compose

Created Docker Compose configuration to orchestrate Peppermint application and PostgreSQL database containers.

**Create project directory and compose file:**
```bash
mkdir ~/peppermint
cd ~/peppermint
nano docker-compose.yml
```

**Docker Compose configuration:**
```yaml
version: "3.1"

services:
  peppermint_postgres:
    container_name: peppermint_postgres
    image: postgres:latest
    restart: always
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: peppermint
      POSTGRES_PASSWORD: 1234
      POSTGRES_DB: peppermint

  peppermint:
    container_name: peppermint
    image: pepperlabs/peppermint:latest
    ports:
      - 3000:3000
      - 5003:5003
    restart: always
    depends_on:
      - peppermint_postgres
    environment:
      DB_USERNAME: "peppermint"
      DB_PASSWORD: "1234"
      DB_HOST: "peppermint_postgres"
      SECRET: 'peppermint4life'

volumes:
  pgdata:
```

**Verify configuration:**
```bash
cat docker-compose.yml
```

**Start containers in detached mode:**
```bash
sudo docker-compose up -d
```

**Verify containers are running:**
```bash
sudo docker ps
```

Both `peppermint` and `peppermint_postgres` containers showed status "Up".

### 3. Access Peppermint Web Interface

Accessed Peppermint via web browser on host machine.

**URL:** `http://192.168.138.12:3000`

**Default credentials:**
- Username: `admin@admin.com`
- Password: `1234`

Successfully logged in and presented with Peppermint dashboard.

**Security note:** Changed default admin password immediately after first login via Settings > Users.

### 4. Configure Users and Clients

...

## What I Learned
### Docker Container Orchestration:

**Multi-container dependencies:** Learnt how Docker Compose orchestrates multiple containers with dependencies. The depends_on directive ensured the PostgreSQL database container started before the Peppermint application container, preventing connection failures during startup.

**Container networking:** Discovered that containers in the same Docker Compose project can communicate using service names as hostnames. The Peppermint container connects to PostgreSQL using DB_HOST: "peppermint_postgres" rather than an IP address, as Docker Compose creates an internal network automatically.

### Security Best Practices:

**Default credentials vulnerability:** Experienced firsthand why default credentials are a security risk. The system shipped with `admin@admin.com:1234`, which would be trivial for attackers to exploit. Changing these immediately after deployment is essential.

### Overall Growth:
This project bridged the gap between theoretical Docker knowledge and practical deployment. I moved from simply understanding what containers are to actually orchestrating a multi-container application with persistent storage and network communication. The troubleshooting experiences taught me systematic debugging approaches that apply beyond just Docker.

## Troubleshooting
### Issue 1: Docker installation fails due to conflicting packages
**Symptoms:**** Installation errors when attempting to install Docker from the official repository

**Cause:** Conflicting unofficial Docker packages from Ubuntu repositories are already installed on the system

**Solution:** Remove all unofficial Docker packages before installing from the official Docker repository:

```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
Then follow installation steps from Step 1.

### Issue 2: Peppermint container won't start or crashes
**Symptoms:** Container fails to start or unexpectedly stops running

**Cause:** Configuration errors, missing dependencies, or resource constraints

**Solution:** Check container logs for specific errors:

```bash
sudo docker logs peppermint
```
Restart the containers:
```bash
sudo docker-compose down
sudo docker-compose up -d
```

### Issue 3: Cannot access Peppermint web interface
**Symptoms:** Unable to reach the Peppermint interface via web browser

**Cause:** Containers not running, firewall blocking port 3000, or incorrect IP address

**Solution:**

1. Verify containers are running:
```bash
sudo docker ps
```
2. Check Ubuntu firewall isn't blocking port 3000:
```bash
sudo ufw status
sudo ufw allow 3000/tcp
```
3. Verify you're accessing the correct IP address (192.168.138.12:3000 in this lab)

## How Can It Be Improved?

- Configure SLA (Service Level Agreement) timers for priority-based response times
- Create knowledge base articles from resolved tickets
- Set up ticket templates for common issue types

## Additional Resources

- [Peppermint Official Documentation](https://docs.peppermint.sh/)
- [Docker Installation Guide for Ubuntu](https://docs.docker.com/engine/install/ubuntu/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [Peppermint GitHub Repository](https://github.com/Peppermint-Lab/peppermint)

## 📹 Demonstration
![Embedded Video](path/to/Video.mp4)