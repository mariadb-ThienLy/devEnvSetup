# Dev Setup

## Overview

This guide provides instructions for setting up a development environment for MaxScale and SuperMax.

## Setup GIT Environment for MaxScale

The current development branch is in this repository:  
https://github.com/mariadb-corporation/MaxScalePriv

Both **MaxScale** and **SuperMax** can be built from the `p_develop` branch.  
`p_25.01` is the branch for the latest major release, located in the **MaxScalePriv** repository.  
Earlier versions of **MaxScale** are in the `MaxScale` repository.

### Merging Flow

```
MaxScale: 21.06 -> 22.08 -> 23.02 -> 23.08 -> 24.02
MaxScalePriv: p_25.01 -> p_develop
```

When something has been done in MaxScale: `24.02 -> p_25.01`

### Initial Setup

1. Create workspace and clone MaxScale:

```bash
mkdir -p "$HOME/workspace" && cd $HOME/workspace && git clone git@github.com:mariadb-corporation/MaxScale.git
```

2. Add MaxScalePriv as a remote:

```bash
cd MaxScale && git remote add p_origin git@github.com:mariadb-corporation/MaxScalePriv.git
```

## Setup Build and Run Scripts

### Clone and Install Scripts

1. Clone this repository:

```bash
cd $HOME/workspace && git clone git@github.com:mariadb-ThienLy/devEnvSetup.git && cd devEnvSetup
```

2. Copy scripts to workspace:

```bash
cp -r script $HOME/workspace && chmod -R +x $HOME/workspace/script
```

3. Add scripts to PATH:

```bash
echo 'PATH=$HOME/workspace/script:$PATH' >> ~/.bashrc && source ~/.bashrc
```

4. Copy required configuration file:

```bash
cp .bashcolors $HOME
```

## Building MaxScale and SuperMax

### Prerequisites

Create your build directory. The directory name must start with `build`:

```bash
mkdir -p "$HOME/workspace/builds/build_dev" && cd $HOME/workspace/builds/build_dev
```

### Run install_build_deps.sh before building

This usually only needs to be done once, unless new dependencies are added.

```bash
cd $HOME/workspace/MaxScale/BUILD && ./install_build_deps.sh
```

### Build Development Version

Build both MaxScale and SuperMax (for `p_develop` branch only):

```bash
maxbuild -S -i $HOME/workspace/maxscale-dev -s $HOME/workspace/MaxScale
```

### Build Specific Versions

To build earlier versions of **MaxScale**:

1. Checkout the desired branch:

```bash
cd $HOME/workspace/MaxScale && git checkout --track origin/24.02.6
```

2. Build the specific version:

```bash
mkdir -p "$HOME/workspace/builds/build_24.02.6" && cd $HOME/workspace/builds/build_24.02.6 && maxbuild -F -i $HOME/workspace/maxscale-24.02.6 -s $HOME/workspace/MaxScale
```

## SSL Certificate Setup

### Install mkcert

Follow the instructions [here](https://github.com/FiloSottile/mkcert?tab=readme-ov-file#installation) to install mkcert.

### Generate Certificates

1. Create certificate directory:

```bash
mkdir -p "$HOME/workspace/cert"
```

2. Generate local SSL certificates:

```bash
mkcert -cert-file $HOME/workspace/cert/cert.pem -key-file $HOME/workspace/cert/key.pem 127.0.0.1 localhost
```

## MaxScale Configuration

### Create Configuration Directory

```bash
mkdir -p "$HOME/workspace/maxconfig/dev" && cd $HOME/workspace/maxconfig/dev
```

### Configuration Files

Create the following configuration files:

#### maxscale.cnf

```ini
[maxscale]
admin_host=127.0.0.1
admin_port=8989
admin_oidc_url=https://127.0.0.1:8990
admin_oidc_client_id=admin
admin_oidc_client_secret=mariadb
admin_oidc_flow=auto
#Replace user with your user dir
admin_ssl_cert=/home/user/cert/cert.pem
admin_ssl_key=/home/user/cert/key.pem
admin_ssl_ca=/home/user/.local/share/mkcert/rootCA.pem

[Monitor]
replication_password=mariadb
replication_user=maxscale
module=mariadbmon
password=mariadb
servers=server_0,server_1,server_2
type=monitor
user=maxscale

[Read-Only-Service]
type=service
router=readconnroute
servers = server_0, server_1, server_2
user = maxscale
password = mariadb

[Read-Only-Listener]
type=listener
service=Read-Only-Service
protocol=mariadbprotocol
port=9908

[Read-Write-Service]
type = service
router = readwritesplit
servers = server_0
user = maxscale
password = mariadb

[Read-Write-Listener]
type=listener
service=Read-Write-Service
protocol=mariadbprotocol
port=9907

[server_0]
type = server
address = 127.0.0.1
port = 4000

[server_1]
type = server
address = 127.0.0.1
port = 4001

[server_2]
type = server
address = 127.0.0.1
port = 4002
```

#### supermax.cnf

```ini
[supermax]
admin_host=127.0.0.1
admin_port=8990
#Replace user with your user dir
admin_ssl_cert=/home/user/cert/cert.pem
admin_ssl_key=/home/user/cert/key.pem
admin_ssl_ca=/home/user/.local/share/mkcert/rootCA.pem
```

## Docker Environment Setup

### Generate Docker Compose Files

Auto-generate docker-compose for primary/replica servers:

```bash
cd $HOME/workspace/maxconfig/dev && make-docker-env.py
```

### Start Containers

Run containers in the background:

```bash
cd $HOME/workspace/maxconfig/dev/docker-env/127.0.0.1 && docker-compose up -d
```

## Running MaxScale and SuperMax

### Run SuperMax

```bash
runmax maxscale-dev dev -S
```

### Run MaxScale

```bash
runmax maxscale-dev dev
```
