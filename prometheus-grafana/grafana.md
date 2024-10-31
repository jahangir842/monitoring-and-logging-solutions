### Grafana with Docker Compose

Setting up Grafana with Docker Compose and a persistent volume will allow for flexible dashboard management and data persistence across container restarts.

---

### **Step 1: Set Up the Directory Structure**

Create a directory for Grafana’s configuration and Docker Compose setup:

```bash
mkdir -p ~/grafana
cd ~/grafana
```

---

### **Step 2: Create Docker Compose File for Grafana**

Inside the `grafana` directory, create a `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  grafana_data:
    driver: local
```

**Explanation**:
- **Image**: Specifies the `grafana/grafana` image, which is the official Grafana Docker image.
- **Container Name**: Names the container `grafana` for easy identification.
- **Ports**: Exposes Grafana on port `3000` to allow access to the Grafana web interface.
- **Volumes**:
  - `grafana_data:/var/lib/grafana`: Creates a persistent Docker volume named `grafana_data` for Grafana's data storage, ensuring all dashboards, settings, and data are saved across container restarts.

---

### **Step 3: Deploy Grafana with Docker Compose**

From within the `~/grafana` directory, run Docker Compose to start Grafana:

```bash
docker-compose up -d
```

This command will:
1. Pull the `grafana/grafana` Docker image (if not already present).
2. Create and initialize the `grafana_data` volume for persistent storage.
3. Start the Grafana container with the specified volume and network configuration.

---

### **Step 4: Access the Grafana Web Interface**

1. **Open Grafana**: In your web browser, go to `http://localhost:3000` (or the server’s IP if accessed remotely).

2. **Log in**: The default credentials are:
   - **Username**: `admin`
   - **Password**: `admin`

3. **Change the Default Password**: On your first login, Grafana will prompt you to change the password. Choose a secure password for future logins.

---

### **Step 5: Verify Data Persistence**

1. **Create a Dashboard**: Set up a sample dashboard in Grafana.
   
2. **Stop and Restart Grafana**:
   - Stop the container with `docker-compose down`.
   - Restart it with `docker-compose up -d`.

3. **Check for Saved Data**: Once Grafana is back up, the previously created dashboard should still be there, verifying that data is saved to the `grafana_data` persistent volume.

---

### **Step 6: Additional Docker Compose Management Commands**

- **Stop Grafana**:

  ```bash
  docker-compose stop
  ```

- **Start Grafana**:

  ```bash
  docker-compose start
  ```

- **Shut Down and Remove Containers** (preserves volume data):

  ```bash
  docker-compose down
  ```

- **Remove Everything, Including Volumes** (deletes saved data):

  ```bash
  docker-compose down -v
  ```

---

### **Summary**

Deploying Grafana with Docker Compose and a persistent volume provides a reliable setup for managing dashboards with saved configurations and data across container restarts.
