Enabling Prometheus and Accessing Metrics in GitLab**

### **Step 1: Ensure You're Using the Correct GitLab Edition**
- **GitLab Community Edition (CE)**: Supports basic Prometheus integration for monitoring GitLab instance metrics.
- **GitLab Enterprise Edition (EE)**: Provides additional features like CI/CD pipeline monitoring and enhanced metrics.

### **Step 2: Access the GitLab Container (If Using Docker)**
If you're running GitLab in a Docker container, follow these steps to access the container and make the necessary changes to the configuration.

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
   In the `gitlab.rb` file, find and ensure the following setting is enabled:

   ```ruby
   prometheus['enable'] = true   # Enable Prometheus monitoring
   prometheus['listen_address'] = '0.0.0.0:9090'  # Expose Prometheus on port 9090
   ```

   - **`prometheus['enable'] = true`**: This enables Prometheus for GitLab to expose metrics.
   - **`prometheus['listen_address'] = '0.0.0.0:9090'`**: This ensures that Prometheus listens on all IP addresses and is accessible externally on port 9090.

3. **Enable IP Allowlisting for Monitoring Endpoints** (Optional but recommended):
   By default, GitLab restricts access to monitoring endpoints (like `/metrics`) using IP allowlisting. You can add specific IPs or IP ranges to allow external access:

   - Add the IP addresses or ranges in the `gitlab.rb` file:

     ```ruby
     gitlab_rails['monitoring_whitelist'] = ['127.0.0.0/8', '192.168.0.1']
     ```

     This allows requests from the specified IP ranges to access the `/metrics` endpoint.

4. **Save the `gitlab.rb` file** after making the changes.

---

### **Step 4: Reconfigure GitLab**

After making changes to the `gitlab.rb` configuration, you need to reconfigure GitLab to apply these settings.

1. **Run the reconfiguration command**:
   ```bash
   sudo gitlab-ctl reconfigure
   ```

   This will apply the changes and restart the necessary GitLab services.

---

### **Step 5: Expose the Necessary Ports**

If you're using **Docker** or **Docker Compose**, ensure you're mapping the correct ports for external access.

1. **For `docker run`**:
   Ensure you expose ports 80, 443 (for GitLab web), and 9090 (for Prometheus metrics):

   ```bash
   docker run -d \
     -p 80:80 \
     -p 443:443 \
     -p 9090:9090 \
     --name gitlab \
     gitlab/gitlab-ce:latest
   ```

2. **For Docker Compose** (`docker-compose.yml`):
   In the `docker-compose.yml` file, ensure the ports are correctly mapped:

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

Once GitLab is reconfigured and the ports are exposed, you can test the Prometheus metrics endpoint:

1. **Locally on the GitLab server** (using `localhost`):
   ```bash
   curl http://localhost:9090/metrics
   ```

2. **Externally (from another machine)**, replace `<your_host_ip_or_domain>` with the IP or domain of your GitLab server:
   ```bash
   curl http://<your_host_ip_or_domain>:9090/metrics
   ```

   If you're using **HTTPS**, use `https://` instead:
   ```bash
   curl https://<your_host_ip_or_domain>:9090/metrics
   ```

---

### **Step 7: Review GitLab Logs (If Issues Persist)**

If you're still encountering issues, check the GitLab logs for any related errors:

1. **View logs** for any errors related to Prometheus or access issues:
   ```bash
   sudo gitlab-ctl tail
   ```

   Look for entries related to Prometheus or access control, and address any specific issues identified.

---

### **Step 8: Monitor and Visualize Metrics in Grafana** (Optional)

If you want to visualize GitLab metrics in **Grafana**:

1. **Add Prometheus as a Data Source in Grafana**:
   - Go to **Grafana** → **Configuration** → **Data Sources** → **Add data source**.
   - Select **Prometheus** and set the URL to your GitLab Prometheus endpoint (e.g., `http://<your_host_ip_or_domain>:9090`).

2. **Create Dashboards** using the available Prometheus metrics from GitLab.

---

## **Summary**

1. **Enable Prometheus** in the `gitlab.rb` configuration file by setting `prometheus['enable'] = true`.
2. **Allow external access** to the metrics endpoint by setting `prometheus['listen_address'] = '0.0.0.0:9090'`.
3. Optionally, **enable IP allowlisting** for monitoring endpoints.
4. **Reconfigure GitLab** to apply the changes.
5. **Expose the necessary ports** for external access to Prometheus metrics.
6. Test access to `/metrics` both locally and externally.
7. Optionally, **integrate with Grafana** for visualization.

By following these steps, you'll have Prometheus enabled on GitLab and be able to access and visualize GitLab metrics, whether for internal monitoring or integration with tools like Grafana.

Let me know if you have any questions or need further clarification!
