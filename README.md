# Hyperledger Fabric Development Environment Setup Guide

## Table of Contents

- [Prerequisites](#prerequisites)
- [Version Requirements](#version-requirements)
- [Installation Steps](#installation-steps)
  - [Step 1: Update System & Install Basic Tools](#step-1-update-system--install-basic-tools)
  - [Step 2: Install Docker & Docker Compose](#step-2-install-docker--docker-compose)
  - [Step 3: Install Go](#step-3-install-go)
  - [Step 4: Install Node.js & Angular](#step-4-install-nodejs--angular)
  - [Step 5: Install Fabric Binaries](#step-5-install-fabric-binaries)
  - [Step 6: Verify Installation](#step-6-verify-installation)
- [Environment Configuration](#environment-configuration)
- [Project Setup & Deployment](#project-setup--deployment)
- [Daily Workflow](#daily-workflow)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

- Ubuntu 24.04 LTS
- Sudo privileges
- Stable internet connection
- MongoDB Atlas account (for database access)

---

## Version Requirements

The following versions are required for compatibility:

| Tool | Version |
|------|---------|
| Docker | 28.2.2 |
| Docker Compose | v5.0.0 |
| Go | 1.21.5 |
| Node.js | 24.11.1 |
| Angular CLI | 21.0.4 |
| Fabric Peer/Orderer | 2.5.0 |
| Fabric CA | 1.5.7 |
| Hyperledger Fabric Images | 2.2.8 |

---

## Installation Steps

> **Important:** Run all installation commands from your home directory (`cd ~`)

### Step 1: Update System & Install Basic Tools

Start by updating your system and installing essential build tools:

```bash
cd ~
sudo apt-get update
sudo apt-get install -y curl git jq build-essential
```

**Note:** Ubuntu 24.04 typically installs jq 1.7 by default. Version 1.6 is also compatible with Fabric.

---

### Step 2: Install Docker & Docker Compose

Install Docker using the official convenience script:

```bash
cd ~

# 1. Download official Docker installer
curl -fsSL https://get.docker.com -o get-docker.sh

# 2. Run the installer
sudo sh get-docker.sh

# 3. Add your user to the docker group (eliminates need for 'sudo' with docker commands)
sudo usermod -aG docker $USER

# 4. Activate the group change (or restart your PC)
newgrp docker
```

**Verify installation:**
```bash
docker --version
docker compose version
```

---

### Step 3: Install Go

Install Go version 1.21.5:

```bash
cd ~

# 1. Download Go 1.21.5
wget https://go.dev/dl/go1.21.5.linux-amd64.tar.gz

# 2. Remove any existing Go installation and install the new version
sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.21.5.linux-amd64.tar.gz

# 3. Add Go to PATH (permanent configuration)
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
```

**Verify installation:**
```bash
go version
```

Expected output: `go version go1.21.5 linux/amd64`

---

### Step 4: Install Node.js & Angular

Use NVM (Node Version Manager) for precise version control:

```bash
cd ~

# 1. Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 2. Load NVM (or restart terminal)
source ~/.bashrc

# 3. Install Node.js 24.11.1
nvm install 24.11.1
nvm use 24.11.1

# 4. Set default Node version
nvm alias default 24.11.1

# 5. Install Angular CLI 21.0.4 globally
npm install -g @angular/cli@21.0.4
```

**Verify installation:**
```bash
node -v
ng version
```

---

### Step 5: Install Fabric Binaries

Install Hyperledger Fabric binaries version 2.5.0:

```bash
cd ~

# 1. Create directory for Fabric samples
mkdir -p fabric-samples
cd fabric-samples

# 2. Download Fabric 2.5.0 and CA 1.5.7
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.5.0 1.5.7

# 3. Add binaries to PATH
echo 'export PATH=$PATH:$HOME/fabric-samples/bin' >> ~/.bashrc
source ~/.bashrc
```

**Verify installation:**
```bash
peer version
orderer version
cryptogen version
```

---

### Step 6: Verify Installation

Run this comprehensive verification check:

```bash
echo "=== VERSION VERIFICATION ==="
echo "Docker: $(docker --version)"
echo "Docker Compose: $(docker compose version)"
echo "Node.js: $(node -v)"
echo "Go: $(go version)"
echo "Angular: $(ng version | grep 'Angular CLI' | head -n 1)"
echo "Peer: $(peer version | grep Version)"
echo "Orderer: $(orderer version | grep Version)"
echo "JQ: $(jq --version)"
```

---

## Environment Configuration

### Configure Bash Environment

Open your bash configuration file:

```bash
nano ~/.bashrc
```

Add or verify the following lines are present:

```bash
# Go configuration
export PATH=$PATH:/usr/local/go/bin
export GOPATH=/home/$USER/go

# Fabric binaries
export PATH=$PATH:$HOME/fabric-samples/bin

# Docker Compose alias
alias docker-compose='docker compose'
```

**Save and apply changes:**
- Press `Ctrl + O` → `Enter` → `Ctrl + X`
- Run: `source ~/.bashrc`

### Set Default Node Version

```bash
nvm alias default 24.11.1
nvm use 24.11.1
```

---

## Project Setup & Deployment

### Initial Setup (Day 1)

1. **Extract and navigate to project:**
   ```bash
   # Unzip your fabric-topologies folder
   cd ~/fabric-topologies/topologies/t1
   ```

2. **Set correct permissions:**
   ```bash
   sudo chown -R $USER:$USER .
   ```

3. **Initialize the network:**
   ```bash
   ./setup-network.sh
   ./deploy-voting-fixed.sh
   ```

4. **Setup application server:**
   ```bash
   cd application/server
   npm install
   ```

5. **Configure MongoDB:**
   - Get your public IP: `curl ifconfig.me`
   - Add this IP to MongoDB Atlas whitelist
   - Update `.env` file with your MongoDB connection string

6. **Set server permissions and start:**
   ```bash
   sudo chown -R $USER:$USER ../../crypto-material
   ./setup-docker-server.sh --fresh
   ```
7. **Run frontend:**
   ```bash
   cd votingAngular
   ng serve
   ```
   

### Environment Variables

Check your `.env` file in the fabric project folder:

```bash
COMPOSE_PROJECT_NAME=hl-fabric-topology-t1
FABRIC_CA_VERSION=1.5
FABRIC_PEER_VERSION=2.2.8
FABRIC_TOOLS_VERSION=2.2.8
PEER_ORDERER_VERSION=2.2.8
```

---

## Daily Workflow

### Starting Work (Day 2 onwards)

Resume your existing network with preserved data:

```bash
cd ~/fabric-topologies/topologies/t1

./resume-network.sh
./resume-voting-server.sh
```

### During Development

When you modify `app.js` or other application files:

```bash
cd application/server
./setup-docker-server.sh
```

### Ending Your Work Session

Preserve data and stop containers:

```bash
./stop-network.sh
```

### Complete Reset (Use with Caution)

> **Warning:** This will delete ALL network data and configurations!

```bash
./teardown-network.sh
```

---

## Troubleshooting

### MongoDB Connection Error

If you encounter MongoDB connection errors:

1. Get your public IP:
   ```bash
   curl ifconfig.me
   ```

2. Add this IP to MongoDB Atlas Network Access whitelist

3. Verify connection string in `application/server/.env`

### Permission Issues

If you encounter permission errors:

```bash
sudo chown -R $USER:$USER ~/fabric-topologies
```

### Docker Group Not Active

If docker commands require sudo:

```bash
newgrp docker
# Or logout and login again
```

### NVM Command Not Found

After installing NVM, reload your shell configuration:

```bash
source ~/.bashrc
```

Or restart your terminal.

---

## Expected Running Containers

When your network is fully operational, you should see containers similar to:

```
voting-server
dev-t1-org3-peer1-voting_1.0-*
dev-t1-org2-peer1-voting_1.0-*
dev-t1-org3-peer1-myccv1-*
dev-t1-org2-peer1-myccv1-*
t1-org1-orderer1/2/3
t1-org2/org3-peer1/2
t1-org2/org3-cli-peer1
t1-org1/org2/org3-ca-identities
t1-org1/org2/org3-ca-tls
```

Check with:
```bash
docker ps --format "table {{.Names}}\t{{.Image}}"
```

---
## Comparison(AI laptop vs Server PC)
![Screenshot from 2026-02-09 14-04-41](https://hackmd.io/_uploads/SykzaEDPWl.png)
![Screenshot from 2026-02-09 14-09-13](https://hackmd.io/_uploads/rysGp4PDbl.png)
![Screenshot from 2026-02-09 14-10-03](https://hackmd.io/_uploads/HJmX6NPwZx.png)


## Summary



For questions or issues, refer to the [Hyperledger Fabric Documentation](https://hyperledger-fabric.readthedocs.io/).

---

*Last Updated: February 2026*
