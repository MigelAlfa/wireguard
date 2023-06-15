# How to deploy

1) Install docker and docker-compose

    sudo curl -L "https://github.com/docker/compose/releases/download/v2.6.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose && sudo chmod +x /usr/local/bin/docker-compose && sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose && docker-compose --version

    curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh

2) create user developer, add user developer to docker group, give docker-compose to docker group, allow docker group users to execute it

    usermod -aG docker developer
    chgrp docker /usr/local/bin/docker-compose
    chmod 750 /usr/local/bin/docker-compose

3) Login to developer user bash

    su developer

4) Generate ssh keys and copy public key ~/.ssh/id_rsa.pub into bitbucket

    ssh-keygen

5) Go to user directory

    cd /home/developer

6) Create docker folder

    mkdir -p docker && cd docker

7) Clone the code from repo

    git clone git@bitbucket.org:d09e3b9e7f4f/docker_wireguard.git

8) Create .env file and setup values

    cp .env.example .env

9) Create wireguard container

    docker-compose up -d

10) Open browser ip:51821, login using password from docker-compose.yml

11) Create and download client's condif files.

## Loadbalancer with two wire servers or simultaneous connection from client to two wire servers

You should setup this repo to two servers on the different WG_PORT and different internal network:

### Example

First server:

```bash
WG_PORT=51111

WG_DEFAULT_ADDRESS=10.8.0.x

WG_POST_UP="iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE; iptables -A INPUT -p udp -m udp --dport 51111 -j ACCEPT; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT"

WG_POST_DOWN="iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE; iptables -D INPUT -p udp -m udp --dport 51111 -j ACCEPT; iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT"
```

Second server:

```bash
WG_PORT=52222

WG_DEFAULT_ADDRESS=10.8.1.x

WG_POST_UP="iptables -t nat -A POSTROUTING -s 10.8.1.0/24 -o eth0 -j MASQUERADE; iptables -A INPUT -p udp -m udp --dport 52222 -j ACCEPT; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT"

WG_POST_DOWN="iptables -t nat -D POSTROUTING -s 10.8.1.0/24 -o eth0 -j MASQUERADE; iptables -D INPUT -p udp -m udp --dport 52222 -j ACCEPT; iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT"
```

Client will have two configs wg0.conf and wg1.conf

### Wire tips and doc <https://github.com/pirate/wireguard-docs#Table>
