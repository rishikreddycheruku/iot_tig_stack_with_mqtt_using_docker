
---

# 🌐 IoT TIG Stack with MQTT using Docker

This project sets up a complete IoT monitoring stack using Docker Compose. It includes the following services:

- **InfluxDB 2.7** – Time-series database to store sensor data.
- **Eclipse Mosquitto** – Lightweight MQTT broker.
- **Telegraf** – Agent to collect data from MQTT and send it to InfluxDB.
- **Grafana** – Visualization tool to display metrics from InfluxDB.

---

## 📦 Prerequisites

- Docker installed: [Install Docker](https://docs.docker.com/get-docker/)
- Docker Compose installed: [Install Docker Compose](https://docs.docker.com/compose/install/)

---

## 📁 Project Structure

```
.
├── docker-compose.yml
├── telegraf.conf
└── mosquitto.conf
```

---

## 🧰 Step-by-Step Setup

### 1. Clone the Repository

```bash
git clone https://github.com/rishikreddycheruku/iot_tig_stack_with_mqtt_using_docker.git
cd iot_tig_stack_with_mqtt_using_docker
```

---

### 2. Configure Required Files as per the Requirement

#### `mosquitto.conf`

```conf
persistence true
persistence_location /mosquitto/data/
log_dest stdout
listener 1883
allow_anonymous true
```

#### `telegraf.conf`

```toml
[agent]
  interval = "10s"
  round_interval = true

[[outputs.influxdb_v2]]
  urls = ["http://influxdb:8086"]
  token = "mysecrettoken123"
  organization = "my-org"
  bucket = "iotdb"

[[inputs.mqtt_consumer]]
  servers = ["tcp://mosquitto:1883"]
  topics = ["iot/sensor"]
  data_format = "json"
  name_override = "esp32_data"
```

---

### 3. Run Docker Compose

```bash
docker-compose up -d
```

This command will:

- Pull the required images.
- Start all services: InfluxDB, Mosquitto, Telegraf, and Grafana.
- Set up InfluxDB with admin credentials and bucket.
- Configure MQTT broker and telemetry pipeline.

---

## 🔌 Service Access

| Service     | URL                           | Default Credentials |
|-------------|-------------------------------|---------------------|
| InfluxDB    | http://localhost:8086         | admin / admin1234   |
| Grafana     | http://localhost:3000         | admin / admin       |
| Mosquitto   | MQTT broker at localhost:1883 | -                   |

---

## 📤 Sending Data to MQTT

You can publish a test message using `mosquitto_pub`:

```bash
mosquitto_pub -h localhost -t iot/sensor -m '{"temperature":25.4, "humidity":60}'
```

Or using Python:

```python
import paho.mqtt.publish as publish

publish.single(
    topic="iot/sensor",
    payload='{"temperature":25.4, "humidity":60}',
    hostname="localhost"
)
```

---

## 📊 Creating Grafana Dashboard

1. Open Grafana: http://localhost:3000
2. Login with `admin / admin`
3. Add a new **Data Source** → Choose **InfluxDB**
4. Set:
   - URL: `http://influxdb:8086`
   - Token: `mysecrettoken123`
   - Org: `my-org`
   - Default Bucket: `iotdb`
5. Save & test.
6. Create dashboards with measurements like `esp32_data` (temperature, humidity, etc.)

---

## 🧹 Stop and Clean Up

```bash
docker-compose down
```

To remove volumes (be careful — this deletes all data):

```bash
docker-compose down -v
```

---

## 📬 Troubleshooting

- **Telegraf not connecting to MQTT**: Ensure Mosquitto is running and listening on port 1883.
- **InfluxDB setup issues**: Check logs using `docker logs influxdb`
- **Permission issues on volume mounts**: Ensure the config files are readable.

---

## ✅ All Done!

You now have a working IoT data pipeline using MQTT, Telegraf, InfluxDB, and Grafana. 🎉

---
