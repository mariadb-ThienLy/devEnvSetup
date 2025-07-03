## Install docker, docker compose (Ubuntu)

### Update system

sudo apt update && sudo apt upgrade -y

### Install prerequisites

```
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
```

### Add Docker GPG key

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### Add Docker repo

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Update package list

```
sudo apt update
```

### Install Docker Engine

```
sudo apt install -y docker-ce docker-ce-cli containerd.io
```

### Verify Docker service status

```
sudo systemctl status docker --no-pager
```

### Test Docker installation

```
sudo docker run hello-world
```

### Install Docker Compose (replace version if needed)

```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.38.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

### Make Docker Compose executable

```
sudo chmod +x /usr/local/bin/docker-compose
```

### Verify Docker Compose installation

```
docker-compose --version
```

### Add current user to docker group for non-root usage

```
sudo usermod -aG docker $USER
```

Log out and back in or run 'newgrp docker' to use Docker as a non-root user.
