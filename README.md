# Dev Setup

- [Overview](#overview)
- [Setup Git Environment for MaxScale](#setup-git-environment-for-maxscale)
  - [Merging Flow](#merging-flow)
  - [Initial Setup](#initial-setup)
  - [Add ReviewBoard Configuration](#add-reviewboard-configuration)
    - [Setup ReviewBoard Configuration](#setup-reviewboard-configuration)
    - [Post Commits for Review](#post-commits-for-review)
- [Setup Build and Run Scripts](#setup-build-and-run-scripts)
  - [Clone and Install Scripts](#clone-and-install-scripts)
- [Building MaxScale and SuperMax](#building-maxscale-and-supermax)
  - [Prerequisites](#prerequisites)
  - [Install Dependencies Required to Build MaxScale on Your OS](#install-dependencies-required-to-build-maxscale-on-your-os)
  - [Build Development Version](#build-development-version)
  - [Build Specific Versions](#build-specific-versions)
- [SSL Certificate Setup](#ssl-certificate-setup)
  - [Install mkcert](#install-mkcert)
  - [Generate Certificates](#generate-certificates)
- [MaxScale Configuration](#maxscale-configuration)
  - [Create Configuration Directory](#create-configuration-directory)
  - [Configuration Files](#configuration-files)
    - [maxscale.cnf](#maxscalecnf)
    - [supermax.cnf](#supermaxcnf)
- [Docker Environment Setup](#docker-environment-setup)
  - [Generate Docker Compose Files](#generate-docker-compose-files)
  - [Start Containers](#start-containers)
- [Running MaxScale and SuperMax](#running-maxscale-and-supermax)
  - [Run SuperMax](#run-supermax)
  - [Run MaxScale](#run-maxscale)

## Overview

This guide provides instructions for setting up a development environment for MaxScale and SuperMax.

## Setup Git Environment for MaxScale

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

3. Add a pre-push script:
First clone this devEnvSetup repository.
```bash
cd $HOME/workspace/ && git clone git@github.com:mariadb-ThienLy/devEnvSetup.git
```

This helps prevent pushing the `p_develop` branch to https://github.com/mariadb-corporation/MaxScale

```bash
cp $HOME/workspace/devEnvSetup/pre-push $HOME/workspace/MaxScale/.git/hooks
```

```bash
cd $HOME/workspace/MaxScale.git/hooks && chmod +x pre-push
```

### Add ReviewBoard Configuration

We are using [RBCommons](https://rbcommons.com/s/MariaDB/) as a way to review our commits.

The workflow with RBCommons is as follows:

1. Create a commit in your local repository.
2. Post that commit to RBCommons and wait for comments.
3. If there are issues to address, modify the latest commit with `git commit --amend` and go back to step 2.
4. When everything is approved, push the change from your local repository to the GitHub repository.

**Installation guide:** https://www.reviewboard.org/downloads/rbtools/

#### Setup ReviewBoard Configuration

Copy the `.reviewboardrc` file as a local ReviewBoard configuration to the MaxScale repository:

```bash
cp .reviewboardrc $HOME/workspace/MaxScale
```

#### Post Commits for Review

To post your latest commit from the local MaxScalePriv repository for review:

```bash
rbt post --repository=MaxScalePriv --tracking-branch p_origin/p_develop --branch p_develop HEAD
```

To add your latest commit to an existing review request:

```bash
rbt post --repository=MaxScalePriv --tracking-branch p_origin/p_develop --branch p_develop -r 25795 HEAD
```

Where `25795` is the review request number (e.g., https://rbcommons.com/s/MariaDB/r/25795/).

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

### Install Dependencies Required to Build MaxScale on Your OS

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
# Replace user with your user directory
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
servers=server_0,server_1,server_2
user=maxscale
password=mariadb

[Read-Only-Listener]
type=listener
service=Read-Only-Service
protocol=mariadbprotocol
port=9908

[Read-Write-Service]
type=service
router=readwritesplit
servers=server_0
user=maxscale
password=mariadb

[Read-Write-Listener]
type=listener
service=Read-Write-Service
protocol=mariadbprotocol
port=9907

[server_0]
type=server
address=127.0.0.1
port=4000

[server_1]
type=server
address=127.0.0.1
port=4001

[server_2]
type=server
address=127.0.0.1
port=4002
```

#### supermax.cnf

```ini
[supermax]
admin_host=127.0.0.1
admin_port=8990
# Replace user with your user directory
admin_ssl_cert=/home/user/cert/cert.pem
admin_ssl_key=/home/user/cert/key.pem
admin_ssl_ca=/home/user/.local/share/mkcert/rootCA.pem
```

## Docker Environment Setup

To run a cluster of servers in the local environment, we use `docker-compose`.
Check out the installation guide [here](./Docker-Installation.md).

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
