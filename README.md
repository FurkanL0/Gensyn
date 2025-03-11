# Gensyn

![1500x500](https://github.com/user-attachments/assets/83bd93ea-dd7c-4aaf-a4bd-6f3cb7fe9284)


| X        | Minimum              |
|------------------|----------------------------|
| **CPU**          | ARM64 |
| **RAM**          | 16 GB                     |
| **Storage**      | X GB SDD                   |
| **Network**      | 1000 Mbps (1 Gbps+ recommended) |

| Server Provider        | Link              | Features |
|------------------|----------------------------|----------------------------|
| **NetCup**          | [Link](https://www.netcup.com/en/?ref=261820) | Cheap / Paypal |
| **Hetzner**      | [Link](https://hetzner.cloud/?ref=ASjlHtRt2swV)                  | Cheap / Crypto Payment |


# SETUP

## 1. Server Update : 

```bash
sudo apt update -y && sudo apt upgrade -y
```
## 2. Install Packages:

```bash
sudo apt install htop ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev tmux iptables curl nvme-cli git wget make jq libleveldb-dev build-essential pkg-config ncdu tar clang bsdmainutils lsb-release libssl-dev libreadline-dev libffi-dev jq gcc screen unzip lz4 gnupg -y
```

## 3. Docker : 

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```bash
echo "deb [arch=arm64 signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
```

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo systemctl enable --now docker
```

#### Docker Test : 

```bash
sudo docker run hello-world
```

```bash
sudo usermod -aG docker $USER
newgrp docker
```

## Docker Instructions ; 

#### With GPU 

```bash
docker run --gpus all --pull=always -it --rm europe-docker.pkg.dev/gensyn-public-b7d9/public/rl-swarm:v0.0.1 ./run_hivemind_docker.sh
```

#### NO GPU 

```bash
docker run --pull=always -it --rm europe-docker.pkg.dev/gensyn-public-b7d9/public/rl-swarm:v0.0.1 ./run_hivemind_docker.sh
```

![image](https://github.com/user-attachments/assets/68601afb-5171-40cf-803e-0a3e8bfc32a6)


<p align="center">
  <img src="https://komarev.com/ghpvc/?username=FurkanL0&style=flat-square&color=red&label=Profile+Views+/+Repo+Views+" alt="Repo / Profile Views" />
</p>
