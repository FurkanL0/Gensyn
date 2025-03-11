# Gensyn

![1500x500](https://github.com/user-attachments/assets/83bd93ea-dd7c-4aaf-a4bd-6f3cb7fe9284)


| X        | Minimum              |
|------------------|----------------------------|
| **CPU**          | ARM64 |
| **RAM**          | 16 GB                     |
| **Storage**      | X GB SDD                   |
| **Network**      | 1000 Mbps (1 Gbps+ recommended) |
- GPU (Optional): Supported CUDA devices for enhanced performance:
    - RTX 3090
    - RTX 4090
    - A100
    - H100
-  **Note**: You can run the node without a GPU using CPU-only mode (details in the docker-compose.yaml section).

- Netcup ARM64 G11 Kullanım : 

![image](https://github.com/user-attachments/assets/c191972d-39b0-413d-88a9-f3e6d125b792)


| Server Provider        | Link              | Features |
|------------------|----------------------------|----------------------------|
| **NetCup**          | [Link](https://www.netcup.com/en/?ref=261820) | Cheap / Paypal |
| **Hetzner**      | [Link](https://hetzner.cloud/?ref=ASjlHtRt2swV)                  | Cheap / Crypto Payment |


# SETUP

## 1. Server Güncelleme : 

```bash
sudo apt update -y && sudo apt upgrade -y
```
## 2. Paketleri İndirmece:

```bash
sudo apt install htop ca-certificates zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev tmux iptables curl nvme-cli git wget make jq libleveldb-dev build-essential pkg-config ncdu tar clang bsdmainutils lsb-release libssl-dev libreadline-dev libffi-dev jq gcc screen unzip lz4 gnupg -y
```

**3. Dockeri İndirmece**
```bash
# Remove old Docker installations
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Add Docker repository
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test Docker
sudo docker run hello-world
```
* Docker'a Kullanıcıları Ekleme : 
```bash
sudo usermod -aG docker $USER
```

**4. Python İndirmece**
```bash
sudo apt-get install python3 python3-pip
```

---

## Dosyayı Çekiyoruz Server'a 
```bash
git clone https://github.com/gensyn-ai/rl-swarm/
cd rl-swarm
```

---

## `docker-compose.yaml` Dosyası Oluşturuyoruz.
This file defines the services: the RL Swarm node, telemetry collector, and web UI.
1. Eski Dosyaya  İsim Veriyoruz:
```bash
mv docker-compose.yaml docker-compose.yaml.old
```
2. Yeni Dosya Oluşturuyoruz:
```bash
nano docker-compose.yaml
```

3. Sunucuda / PC'de Ekran Kartı Varsa Bunu Yapıştır Yoksa Diğeri :
```bash
version: '3'

services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.120.0
    ports:
      - "4317:4317"  # OTLP gRPC
      - "4318:4318"  # OTLP HTTP
      - "55679:55679"  # Prometheus metrics (optional)
    environment:
      - OTEL_LOG_LEVEL=DEBUG

  swarm_node:
    image: europe-docker.pkg.dev/gensyn-public-b7d9/public/rl-swarm:v0.0.1
    command: ./run_hivemind_docker.sh
    runtime: nvidia  # Enables GPU support; remove if no GPU is available
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - PEER_MULTI_ADDRS=/ip4/38.101.215.13/tcp/30002/p2p/QmQ2gEXoPJg6iMBSUFWGzAabS2VhnzuS782Y637hGjfsRJ
      - HOST_MULTI_ADDRS=/ip4/0.0.0.0/tcp/38331
    ports:
      - "38331:38331"  # Exposes the swarm node's P2P port
    depends_on:
      - otel-collector

  fastapi:
    build:
      context: .
      dockerfile: Dockerfile.webserver
    environment:
      - OTEL_SERVICE_NAME=rlswarm-fastapi
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - INITIAL_PEERS=/ip4/38.101.215.13/tcp/30002/p2p/QmQ2gEXoPJg6iMBSUFWGzAabS2VhnzuS782Y637hGjfsRJ
    ports:
      - "8080:8000"  # Maps port 8080 on the host to 8000 in the container
    depends_on:
      - otel-collector
      - swarm_node
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/api/healthz"]
      interval: 30s
      retries: 3
```
* **GPU/CPU Note**: If you don't have an NVIDIA GPU or the NVIDIA Container Runtime, remove the `runtime: nvidia` line under `swarm_node` to run on **CPU**.

- CPU : 

```bash
version: '3'

services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:0.120.0
    ports:
      - "4317:4317"  # OTLP gRPC
      - "4318:4318"  # OTLP HTTP
      - "55679:55679"  # Prometheus metrics (optional)
    environment:
      - OTEL_LOG_LEVEL=DEBUG

  swarm_node:
    image: europe-docker.pkg.dev/gensyn-public-b7d9/public/rl-swarm:v0.0.1
    command: ./run_hivemind_docker.sh
    environment:
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - PEER_MULTI_ADDRS=/ip4/38.101.215.13/tcp/30002/p2p/QmQ2gEXoPJg6iMBSUFWGzAabS2VhnzuS782Y637hGjfsRJ
      - HOST_MULTI_ADDRS=/ip4/0.0.0.0/tcp/38331
    ports:
      - "38331:38331"  # Exposes the swarm node's P2P port
    depends_on:
      - otel-collector

  fastapi:
    build:
      context: .
      dockerfile: Dockerfile.webserver
    environment:
      - OTEL_SERVICE_NAME=rlswarm-fastapi
      - OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4317
      - INITIAL_PEERS=/ip4/38.101.215.13/tcp/30002/p2p/QmQ2gEXoPJg6iMBSUFWGzAabS2VhnzuS782Y637hGjfsRJ
    ports:
      - "8080:8000"  # Maps port 8080 on the host to 8000 in the container
    depends_on:
      - otel-collector
      - swarm_node
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/api/healthz"]
      interval: 30s
      retries: 3
```


### What Each Service Does:
* **otel-collector**: Gathers telemetry data (metrics, traces).
* **swarm_node**: The core RL Swarm node connecting to the network.
* **fastapi**: The web UI dashboard for monitoring.

---

## RL Swarm Node + Web UI Dashboard Başlatma / Çalıştırma 
Star'ı Verelim:
```bash
docker compose up --build -d && docker compose logs -f
```

![image](https://github.com/user-attachments/assets/ac448740-305a-432d-96bd-7b95a834378e)


* **Note**: The first run may take time due to image downloads. Look for this log to confirm your node joined the swarm

![Screenshot_654](https://github.com/user-attachments/assets/56243405-85ca-41ae-8591-2e61631835da)

* **Loglardan Çıkma**: Press `Ctrl+C`

* -d olduğu için arkada çalışmaya devam edecek.

---

## Log Kontrol
* **RL Swarm node Log Kontrol:**
```bash
docker-compose logs -f swarm_node
```

* **Web UI Log Kontrolü:**
```bash
docker-compose logs -f fastapi
```

* **Telemetry Collector Log Kontrolü:**
```bash
docker-compose logs -f otel-collector
```

* All Logs: Use `docker-compose logs -f` without a service name.

---

## Web UI Dashboard Erişim
* VPS: `http://<your-vps-ip>:8080/`
* Local PC: `http://localhost:8080` or `http://0.0.0.0:8080`

![image](https://github.com/user-attachments/assets/5be7755d-bcc9-41d8-ae03-37816002e014)

## Monitoring Your Node
* The dashboard displays *collective swarm data*, not individual node stats. To track your node:

  1- Check the `swarm_node` logs for your node’s unique ID (e.g., `[F-d2042cff-01c9-4801-8ea7-1c1afc29c9b6]`):
  
![image](https://github.com/user-attachments/assets/4bc5efa2-c9c3-4bf0-8dab-d21069c89a79)

  2- Search for this ID in the dashboard data to see your node’s contributions.

<p align="center">
  <img src="https://komarev.com/ghpvc/?username=FurkanL0&style=flat-square&color=red&label=Profile+Views+/+Repo+Views+" alt="Repo / Profile Views" />
</p>
