

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

<img width="1400" height="1150" alt="image" src="https://github.com/user-attachments/assets/41e7c407-8e94-40da-a081-656bfdd08d9c" />


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

<img width="1204" height="408" alt="image" src="https://github.com/user-attachments/assets/fae5f811-a1c1-4b4c-956f-2a97a6fdb8e2" />


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
<img width="1390" height="759" alt="image" src="https://github.com/user-attachments/assets/a2d7e086-10e8-4e95-b8ac-c0397723b40b" />



---

### 7) Interfaces

* **Prometheus** : <img width="1832" height="432" alt="image" src="https://github.com/user-attachments/assets/649748de-ea87-4828-a693-793ca1fc07b0" />

* **Grafana** : <img width="1824" height="529" alt="image" src="https://github.com/user-attachments/assets/54c97e67-f918-4a72-8799-c409a23d1205" />


  * **Login** par défaut : `admin / admin`

---

## EXPLOITATION DE LA SOLUTION

---

### 1 Ajout de la source Prometheus dans Grafana

> Naviguez dans Grafana → `Settings` → `Data Sources` → `Add data source` → Prometheus
> La connexion doit être validée automatiquement.

<img width="1629" height="725" alt="image" src="https://github.com/user-attachments/assets/b2b422b7-20ed-409d-9865-9f1dae853ed7" />


---

### 2️ Création d’un Dashboard personnalisé

#### Exemple de panels :

---

### Panel 1 : CPU Utilization

* **PromQL** :

```promql
avg without(cpu) (irate(node_cpu_seconds_total{mode!="idle"}[1m]))
```

<img width="1447" height="758" alt="image" src="https://github.com/user-attachments/assets/f9a7f741-4b45-41c1-8f20-9b40033f375f" />


## CONCLUSION

Cette solution permet une **supervision en temps réel** d’un hôte Linux, avec :

> Des **alertes personnalisées**
> Une **interface web intuitive**
> Un **déploiement rapide via Docker**

---

### OUSMANE KA
