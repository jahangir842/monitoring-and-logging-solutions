---

## Prometheus

This guide covers setting up Prometheus### Detailed Notes on Installing Prometheus with Docker Compose and Persistent Volume

Prometheus can be deployed using Docker Compose for easier management and scaling. Setting up a persistent volume ensures that Prometheus data remains intact, even if the container is restarted or removed.

---

### **Step 1: Set Up the Directory Structure**

Create a directory to hold the Prometheus configuration file and Docker Compose setup:

```bash
mkdir -p ~/prometheus
cd ~/prometheus
```

---

### **Step 2: Create Prometheus Configuration File**

Inside the `prometheus` directory, create a `prometheus.yml` file with the following configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

This configuration:
- Sets the global scrape interval to `15s` (default interval Prometheus uses to scrape metrics).
- Defines a scrape job with `localhost:9090` as the target, which allows Prometheus to monitor itself.

---

### **Step 3: Create Docker Compose File**

In the same directory, create a `docker-compose.yml` file to define the Prometheus service with persistent storage:

```yaml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - prometheus_data:/prometheus
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

volumes:
  prometheus_data:
    driver: local
```

**Explanation:**
- **Image**: Uses the official `prom/prometheus` Docker image.
- **Container Name**: Names the container `prometheus`.
- **Ports**: Maps port `9090` of the host to `9090` on the container to allow external access to Prometheus.
- **Volumes**:
  - `prometheus_data:/prometheus`: Creates a named volume `prometheus_data` for persistent storage of Prometheus data.
  - `./prometheus.yml:/etc/prometheus/prometheus.yml`: Binds the `prometheus.yml` configuration file to the Prometheus container.

---

### **Step 4: Deploy Prometheus with Docker Compose**

From within the `~/prometheus` directory, start the Docker Compose service:

```bash
docker-compose up -d
```

**Flags**:
- `-d`: Runs the container in detached mode (in the background).

Docker Compose will:
1. Pull the `prom/prometheus` image if not already available.
2. Create and initialize the `prometheus_data` volume for persistent data storage.
3. Start the Prometheus container using the specified configuration and volume settings.

---

### **Step 5: Verify Prometheus Setup**

1. **Check Docker Logs**:
   To ensure that Prometheus is running without errors, view the container logs:

   ```bash
   docker-compose logs prometheus
   ```

2. **Access Prometheus**:
   Open a browser and go to `http://localhost:9090` (or the IP address of your host machine if accessed remotely).

3. **Check Data Persistence**:
   - Prometheus will store its data in the `prometheus_data` volume.
   - Verify that the data is being persisted by stopping and removing the container:

     ```bash
     docker-compose down
     docker-compose up -d
     ```

   After restarting, all previously collected metrics should remain accessible since they are stored in the persistent volume.

---

### **Step 6: Additional Commands for Docker Compose Management**

- **Stop the Prometheus Container**:

  ```bash
  docker-compose stop
  ```

- **Start the Prometheus Container**:

  ```bash
  docker-compose start
  ```

- **Remove Prometheus Containers and Networks** (does not remove volumes, preserving data):

  ```bash
  docker-compose down
  ```

- **Remove Everything Including Volumes** (deletes stored data):

  ```bash
  docker-compose down -v
  ```

---

### **Summary**

Using Docker Compose with a persistent volume for Prometheus setup ensures that all metrics data is saved to a local directory or Docker volume. This approach provides easy deployment, scaling, and data preservation across container restarts. and related Docker images on an offline Ubuntu system by first downloading the necessary Docker images on an online system.

---

### **Step 1: Download Docker Images on an Online System**

Pull the necessary Docker images using the following commands:

```bash
docker pull prom/prometheus
docker pull grafana/grafana
docker pull gitlab/gitlab-ce
```

---

### **Step 2: Save Docker Images as .tar Files**

Save each Docker image to a `.tar` file so they can be transferred:

```bash
docker save -o prometheus.tar prom/prometheus
docker save -o grafana.tar grafana/grafana
docker save -o gitlab.tar gitlab/gitlab-ce
```

---

### **Step 3: Transfer Images to the Offline System**

Copy the saved `.tar` files (`prometheus.tar`, `grafana.tar`, etc.) from the online system to the offline Ubuntu system. Use a USB drive, SSH, or other file transfer methods available.

---

### **Step 4: Load Docker Images on the Offline System**

On the offline system, load the images from the `.tar` files:

```bash
docker load -i prometheus.tar
docker load -i grafana.tar
docker load -i gitlab.tar
```

---

### **Step 5: Run Prometheus Container on the Offline System**

#### 1. Create a Prometheus Configuration File

Create a `prometheus.yml` file with the following content:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

Save this file as `prometheus.yml` in your current directory.

---

#### 2. Run the Prometheus Docker Container

Use this command to start the Prometheus container with the configuration file:

```bash
docker run -d \
  -p 9090:9090 \
  -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml \
  --name prometheus \
  prom/prometheus
```

This command:
- Runs the container in detached mode (`-d`).
- Exposes Prometheus on port 9090 (`-p 9090:9090`).
- Mounts the `prometheus.yml` configuration file into the container (`-v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml`).
- Assigns the container a name (`--name prometheus`).

---

### **Command Summary**

**On Online System:**

```bash
docker pull prom/prometheus
docker save -o prometheus.tar prom/prometheus
# Transfer prometheus.tar to the offline system
```

**On Offline System:**

```bash
docker load -i prometheus.tar
# Ensure prometheus.yml is in the current directory
docker run -d -p 9090:9090 -v $(pwd)/prometheus.yml:/etc/prometheus/prometheus.yml --name prometheus prom/prometheus
```

This procedure sets up Prometheus to run in Docker on an offline Ubuntu system.
