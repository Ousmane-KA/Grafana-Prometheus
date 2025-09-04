

#  Supervision avec Grafana, Prometheus, Node Exporter & Docker

---

## INTRODUCTION

**Prometheus** est un **système de monitoring open source** développé initialement par **SoundCloud**. Il permet de :

* Collecter des **métriques temporelles**
* Stocker et interroger les données via **PromQL**
* Surveiller des **serveurs**, des **containers** Docker/Kubernetes
* Déclencher des **alertes de performance** ou d'incident

**Grafana** est une **plateforme de visualisation open source**. Elle permet de :

* Créer des **dashboards dynamiques**
* Se connecter à différentes sources de données (Prometheus, InfluxDB, etc.)
* Faire du **reporting temps réel**
* Visualiser des **KPIs système, réseau ou applicatif**

---

## PLAN DE MISE EN PLACE

![image](attachment:43096923-d5b0-4e4a-823a-5ae8b462707d\:image.png)

---

## 🅲 INSTALLATION ET CONFIGURATION DE LA STACK

### 1) Spécifications de la machine

* **OS** : Debian 11
* **CPU** : 4 cœurs
* **RAM** : 8 Go
* **Disque** : 50 Go
* **Outils** : `docker` & `docker-compose`

---

### 2) Arborescence du projet

```bash
GRAFANA-PROMETHEUS/
├── docker-compose.yml
├── grafana/
│   ├── data/
│   └── config/
└── prometheus/
    ├── data/
    └── config/
```

---

### 3) Création des dossiers et gestion des droits

```bash
mkdir -p GRAFANA-PROMETHEUS
cd GRAFANA-PROMETHEUS

sudo chown -R 472:472 grafana
sudo chown -R 65534:65534 prometheus
```

---

### 4) Fichier `docker-compose.yml`

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus/config/:/etc/prometheus/
      - ./prometheus/data/:/prometheus/
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--storage.tsdb.path=/prometheus"
    ports:
      - "9090:9090"

  node_exporter:
    image: prom/node-exporter:latest
    container_name: node_exporter
    ports:
      - "9100:9100"

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - ./grafana/data/:/var/lib/grafana
      - ./grafana/config/:/etc/grafana/provisioning/
    depends_on:
      - prometheus
      - node_exporter
```

---

### 5) Fichier `prometheus.yml`

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['node_exporter:9100']
```

---

### 6) Déploiement de la stack

```bash
docker-compose up -d
```

---

### 7) Interfaces

* **Prometheus** : [http://localhost:9090](http://localhost:9090)
* **Grafana** : [http://localhost:3000](http://localhost:3000)

  * **Login** par défaut : `admin / admin`

---

## EXPLOITATION DE LA SOLUTION

---

### 1 Ajout de la source Prometheus dans Grafana

> Naviguez dans Grafana → `Settings` → `Data Sources` → `Add data source` → Prometheus
> La connexion doit être validée automatiquement.

---

### 2️ Création d’un Dashboard personnalisé

#### Exemple de panels :

---

### Panel 1 : CPU Utilization

* **PromQL** :

```promql
avg without(cpu) (irate(node_cpu_seconds_total{mode!="idle"}[1m]))
```

* **Description** : Affiche l’utilisation active du CPU (hors idle)

---

### Panel 2 : Memory Usage

* **PromQL** :

```promql
node_memory_MemTotal_bytes 
- node_memory_MemFree_bytes 
- node_memory_Cached_bytes 
- node_memory_Buffers_bytes
```

* **Autres métriques** :

  * `node_memory_Cached_bytes`
  * `node_memory_Buffers_bytes`
  * `node_memory_MemFree_bytes`

* **Description** : Utilisation mémoire par type (Used, Buffers, Cached, Free)

---

### Panel 3 : Disk Usage (Used)

```promql
node_filesystem_size_bytes - node_filesystem_avail_bytes
```

> Filtrage des volumes : exclusion des loop, tmpfs, nsfs, etc.

---

### Panel 4 : Disk Available

```promql
node_filesystem_avail_bytes
```

> Visualisation de l’espace disque libre par partition

---

### Panel 5 : Network Traffic

* **Receive** :

```promql
irate(node_network_receive_bytes_total[5m])
```

* **Transmit** :

```promql
irate(node_network_transmit_bytes_total[5m])
```

> 📶 Suivi du trafic réseau entrant/sortant par interface

---

## 🧾 CONCLUSION

Cette solution permet une **supervision en temps réel** d’un hôte Linux, avec :

✅ Des **alertes personnalisées**
✅ Une **interface web intuitive**
✅ Un **déploiement rapide via Docker**

---

### OUSMANE KA
