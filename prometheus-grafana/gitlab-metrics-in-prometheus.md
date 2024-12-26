## **Guide: Enabling Prometheus and Accessing Metrics in GitLab**

### **Step 1: Ensure You're Using the Correct GitLab Edition**
- **GitLab Community Edition (CE)**: Supports basic Prometheus integration for monitoring GitLab instance metrics.
- **GitLab Enterprise Edition (EE)**: Provides additional features like CI/CD pipeline monitoring and enhanced metrics.

### **Step 2: Access the GitLab Container (If Using Docker)**
If you're running GitLab in a Docker container, you need to access the container to make configuration changes.

1. **Access the running GitLab container**:
   ```bash
   docker exec -it <container_name> /bin/bash
   ```

   Replace `<container_name>` with the actual name of your GitLab container (e.g., `gitlab`).

2. Once inside the container, you will need to locate and edit the `gitlab.rb` file to enable Prometheus.

---

### **Step 3: Enable Prometheus Integration**

1. **Edit the `gitlab.rb` file**:
   Open the configuration file for GitLab inside the container. This file contains the settings for Prometheus.

   ```bash
   nano /etc/gitlab/gitlab.rb
   ```

2. **Enable Prometheus**:
   In the `gitlab.rb` file, look for the following setting and ensure it's enabled:
   ```ruby
   prometheus['enable'] = true
   prometheus['listen_address'] = '0.0.0.0:9090'  # Expose Prometheus on port 9090
   ```

   - **`prometheus['enable'] = true`**: This enables Prometheus metrics.
   - **`prometheus['listen_address'] = '0.0.0.0:9090'`**: This ensures Prometheus listens on all IP addresses (and is accessible externally on port 9090).

3. **Enable IP Allowlisting for Monitoring Endpoints** (Optional but recommended):
   By default, GitLab might restrict access to monitoring endpoints (like `/metrics`) via IP allowlisting. If you want to allow specific IPs or ranges to access these endpoints:

   - Add the IP addresses or ranges in the `gitlab.rb` file:
     ```ruby
     gitlab_rails['monitoring_whitelist'] = ['127.0.0.0/8', '192.168.0.1']
     ```

   - This allows requests from IPs within the specified ranges to access the `/metrics` endpoint.

4. **Save the `gitlab.rb` file** after making the changes.

---

### **Step 4: Reconfigure GitLab**

After making changes to the `gitlab.rb` configuration, you need to reconfigure GitLab to apply these settings.

1. **Run the reconfiguration command**:
   ```bash
   sudo gitlab-ctl reconfigure
   ```

   This will apply the changes and restart the necessary services.

---

### **Step 5: Expose the Necessary Ports**

For external access to Prometheus metrics, you need to ensure that the correct ports are exposed from your Docker container.

If you're using **Docker** or **Docker Compose**, ensure you're mapping port 9090 to an external port (e.g., `9090:9090` for Prometheus metrics):

1. **For `docker run`**:
   ```bash
   docker run -d \
     -p 80:80 \
     -p 443:443 \
     -p 9090:9090 \
     --name gitlab \
     gitlab/gitlab-ce:latest
   ```

2. **For Docker Compose** (`docker-compose.yml`):
   ```yaml
   version: '3'
   services:
     gitlab:
       image: gitlab/gitlab-ce:latest
       ports:
         - "80:80"
         - "443:443"
         - "9090:9090"
       environment:
         - GITLAB_OMNIBUS_CONFIG="external_url 'http://<your_host_ip_or_domain>'"
       restart: always
   ```

---

### **Step 6: Test Accessing Prometheus Metrics**

Once GitLab is reconfigured and the ports are exposed, test the Prometheus metrics endpoint:

1. **Locally on the GitLab server** (using `localhost`):
   ```bash
   curl http://localhost:9090/metrics
   ```

2. **Externally (from another machine)**, replace `<your_host_ip_or_domain>` with your GitLab server’s IP or domain:
   ```bash
   curl http://<your_host_ip_or_domain>:9090/metrics
   ```

   If you're using **HTTPS**, make sure to use `https://`:
   ```bash
   curl https://<your_host_ip_or_domain>:9090/metrics
   ```

---

### **Step 7: Review GitLab Logs (If Issues Persist)**

If you're still encountering issues or the `/metrics` page is not accessible:

1. **Check GitLab logs** for any related errors:
   ```bash
   sudo gitlab-ctl tail
   ```

2. Look specifically for any entries related to Prometheus, the reverse proxy, or access issues.

---

### **Step 8: Monitor and Visualize Metrics in Grafana** (Optional)

If you want to visualize the metrics in **Grafana**:

1. **Add GitLab’s Prometheus endpoint** to Grafana as a data source:
   - Go to **Grafana** → **Configuration** → **Data Sources** → **Add data source**.
   - Select **Prometheus** and enter the URL of your GitLab Prometheus endpoint (e.g., `http://<your_host_ip_or_domain>:9090`).

2. **Create Dashboards** using the available Prometheus metrics in Grafana.

---

## **Summary**

- Enable **Prometheus integration** in the `gitlab.rb` configuration file.
- Ensure **IP allowlisting** for monitoring endpoints is properly configured (optional but recommended).
- **Reconfigure GitLab** and expose necessary ports for external access.
- Test access to the `/metrics` endpoint locally and externally.
- Optionally, add Prometheus as a data source in **Grafana** to visualize GitLab metrics.

With this setup, you’ll be able to access GitLab's Prometheus metrics externally and integrate them into tools like **Grafana** for enhanced monitoring and visualization.

Let me know if you need more help or details!
