<!-- START doctoc generated TOC please keep comment here to allow auto update -->
# Build on DeFiChain with Docker

This repository describes the setup of a DeFiChain node, an Ocean API and the corresponding DeFi Scan Explorer.

You can use this infrastructure to either use your own node with your DeFiChain Light Wallet, run your own DeFiChain Community Project based on your own node or do other cool stuff with DeFiChain.

This tutorial uses docker as a runtime platform. You could also run all components directly on a server.

<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table of Contents

- [Build on DeFiChain with Docker](#build-on-defichain-with-docker)
  - [Prerequisites](#prerequisites)
  - [Install Docker](#install-docker)
  - [Run a DeFiChain Node](#run-a-defichain-node)
      - [defi.conf Configuration File](#deficonf-configuration-file)
      - [Backup & Restore](#backup--restore)
  - [Run a DeFiChain Ocean API](#run-a-defichain-ocean-api)
      - [Different Databases](#different-databases)
      - [Backup & Restore](#backup--restore-1)
  - [Run a DeFiChain / DeFiScan Explorer](#run-a-defichain--defiscan-explorer)
      - [Run the container](#run-the-container)
  - [Expose Services to the Internet](#expose-services-to-the-internet)
  - [Further Support](#further-support)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
## Prerequisites

Before you can start, please ensure you have the following components available:

- A linux-based server that is exposed to the internet - e.g. you could use netcup
- A DNS domain if you would like to expose services via a DNS name

## Install Docker

Before we can run any DeFiChain component, we need to install docker.

You can therefore use the Docker convenience script which makes the installation quick and easy:

```sh
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

Create a docker network for all DeFiChain components (we call it "ocean"):

docker create network ocean

Afterwards add a new user "defichain" and create a data directory:

```sh
sudo useradd -m defichain
sudo mkdir /home/defichain/defid-data
sudo mkdir /home/defichain/whale-data
sudo chown defichain:defichain /home/defichain/defid-data
sudo chown defichain:defichain /home/defichain/whale-data
```

Check out https://docs.docker.com/engine/install/ubuntu/ for more details.

If you want to use docker compose, you can jump to TL;DR;JustUseCompose

## Run a DeFiChain Node

```sh
docker run -d --rm  \
  -v /home/defichain/defid-data:/data \
  --network=ocean \
  defi/defichain \
  defid \
  -rpcallowip=172.19.0.0/16 \
  -rpcbind=0.0.0.0 \
  -rpcuser=your-user \
  -rpcpassword=your-password
```

Note: The rpcallowip range must be the docker network IP range.

#### defi.conf Configuration File
If you want to change the defi.conf, just locate the file at /home/defichain/data/defi.conf

#### Backup & Restore
You can also easily do backup by making a copy of the /home/defichain/data folder, e.g. cp /home/defichain/defid-data /home/defichain/defid-data-backup

To restore, just stop the container, run cp /home/defichain/defid-data-backup /home/defichain/defid-data and then run the container again.

Check out https://github.com/DeFiCh/ain/blob/master/doc/setup-nodes-docker.md for more details.

## Run a DeFiChain Ocean API

```sh
docker run -d \
  -p 127.0.0.1:8081:3000 \
  -v /mnt/storagespace/whale/data:/data \
  --network=ocean \
  --env WHALE_DATABASE_PROVIDER=level \
  --env WHALE_DATABASE_LEVEL_LOCATION=/data \
  --env WHALE_DEFID_URL=http://your-user:your-password@172.19.0.2:8554 \
  --env WHALE_NETWORK=mainnet \
  ghcr.io/jellyfishsdk/whale-api
```

#### Different Databases

You can choose between the two database types "memory" and "level" (a leveldb). If you choose the level db, you can specify a location where the data is stored. This enables you to take backups of your database.

#### Backup & Restore
You can also easily do backup by making a copy of the /home/defichain/data folder, e.g. `cp /home/defichain/defid-data /home/defichain/defid-data-backup`

To restore, just stop the container, run `cp /home/defichain/defid-data-backup /home/defichain/defid-data` and then run the container again.

## Run a DeFiChain / DeFiScan Explorer

To run a DeFiChain Blockchain Explorer, you have to clone, edit and re-build the [DeFiCh/scan repository](https://github.com/DeFiCh/scan).

I used this approach to build [https://explorer.sternberg-partners.de/](https://explorer.sternberg-partners.de/). You can find the open source repository for the website [here](https://github.com/lukasosterheider/scan).

The following files need to be edited:
- next-sitemap.js --> change URL
- src/components/commons/Breadcrumb.tsx --> change URL
- next.config.js --> add your Ocean API URL to connect-src
- public/_headers --> add your Ocean API URL to connect-src
- src/layouts/contexts/Environment.ts --> add additional environments (optional)
- src/layouts/contexts/NetworkContext.tsx --> add network information if additional environments have been added (optional)
- src/layouts/contexts/WhaleContext.tsx --> add & edit Ocean API endpoints

You can use the Dockerfile (provided in 'lukasosterheider/scan') to build the application.

#### Run the container

```sh
docker run -d -p 127.0.0.1:8082:3000 container-repo/container-name:tag
```

Example for my DeFiChain Blockchain Explorer:

```sh
docker run -d -p 127.0.0.1:8082:3000 lukasosterheider/defiscan-sternberg-partners:0.1.5
```

## Expose Services to the Internet

To expose the - currently only on localhost available - services, you can utilize nginx as a reverse proxy solution. While further documentation might follow, for now please find more information on the below mentioned websites:

- Install and configure nginx: https://fedingo.com/how-to-host-multiple-domains-on-one-server-in-nginx/
- Use let's encrypt & certbot to enable TLS secured traffic: https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/
- Use Amplify for nginx to monitor your traffic: https://amplify.nginx.com/docs/guide-installing-and-managing-nginx-amplify-agent.html

## Further Support

Feel free to contact me on Twitter for further support [@osterheider](https://twitter.com/osterheider).

Buy me a coffee or pizza by donating to (DeFiChain Address):
`df1qqd3eze5lxc04g835zv8nugu6slww6c5l4yzfcc`