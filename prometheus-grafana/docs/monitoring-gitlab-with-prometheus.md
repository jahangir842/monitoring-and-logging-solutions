# Monitor GitLab in Prometheus

### **Step 2: Access the GitLab Container (If Using Docker)**

If you’re running GitLab in a Docker container, follow these steps to access it and configure Prometheus integration.

1. **Access the GitLab container**:
   ```bash
   docker exec -it <container_name> /bin/bash
   ```
   Replace `<container_name>` with the name of your GitLab container (e.g., `gitlab`).

2. Once inside, locate and edit the `gitlab.rb` file to enable Prometheus.

---

### **Step 3: Enable Prometheus Integration**

1. **Backup the `gitlab.rb` file**:  
   Always create a backup before making changes:
   ```bash
   cp /etc/gitlab/gitlab.rb /etc/gitlab/gitlab.rb.bak
   ```

2. **Edit the `gitlab.rb` file**:  
   Open the file in an editor:
   ```bash
   nano /etc/gitlab/gitlab.rb
   ```

3. **Enable Prometheus**:  
   Add or modify the following settings:
   ```ruby
   prometheus['enable'] = true
   prometheus['listen_address'] = '0.0.0.0:9090'
   ```

4. **Optional - Enable IP Allowlisting for Monitoring Endpoints**:  
   Restrict `/metrics` access to specific IPs or ranges:
   ```ruby
   gitlab_rails['monitoring_whitelist'] = ['127.0.0.0/8', '192.168.0.0/16']
   ```

5. Save and close the file.

---

### **Step 4: Reconfigure GitLab**

After updating the `gitlab.rb` file, reconfigure GitLab to apply the changes:

1. Run the following command:
   ```bash
   gitlab-ctl reconfigure
   ```
   This command applies the configuration and restarts GitLab services.

---

### **Step 5: Test Prometheus Metrics Endpoint**

1. **Locally on the GitLab server**:  
   Test access to the metrics endpoint:
   ```bash
   curl http://localhost:9090/-/metrics
   ```

2. **Externally from another machine**:  
   Replace `<your_host_ip_or_domain>` with your GitLab server's IP or domain:
   ```bash
   curl http://<your_host_ip_or_domain>:9090/-/metrics
   ```
   For HTTPS, use:
   ```bash
   curl https://<your_host_ip_or_domain>:9090/-/metrics
   ```

---

### **Step 6: Configure Prometheus to Scrape GitLab Metrics**

Update the Prometheus configuration file (`prometheus.yml`) to include GitLab:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'gitlab'
    metrics_path: '/-/metrics'  # Correct path to GitLab metrics
    static_configs:
      - targets: ['192.168.1.208:9090']  # Replace with your GitLab server's address
```

Reload Prometheus to apply the updated configuration:
```bash
docker restart prometheus
```

---

### **Step 7: Visualize Metrics in Grafana**

Once Prometheus is scraping GitLab metrics, you can use Grafana to create a dashboard for visualizing these metrics.

#### **1. Add Prometheus as a Data Source**
1. Log in to Grafana.
2. Navigate to **Configuration** → **Data Sources**.
3. Click **Add data source** and select **Prometheus**.
4. In the URL field, enter the address of your Prometheus server, such as:
   ```plaintext
   http://<prometheus_host>:9090
   ```
5. Click **Save & Test** to verify the connection.

---

#### **2. Create a New Dashboard**
1. Go to the **Dashboards** menu and click **+ New Dashboard**.
2. Select **Add a New Panel** to create a custom panel for visualizing GitLab metrics.

---

#### **3. Link the Dashboard to GitLab Metrics**
1. In the **Query** section of the panel editor:
   - Choose **Prometheus** as the data source.
   - Write PromQL queries to fetch GitLab-specific metrics, such as:
     - **GitLab process memory usage**:
       ```promql
       process_resident_memory_bytes{job="gitlab"}
       ```
     - **GitLab HTTP requests per second**:
       ```promql
       rate(http_requests_total{job="gitlab"}[1m])
       ```
     - **Sidekiq job queue latency**:
       ```promql
       sidekiq_queue_latency_seconds{job="gitlab"}
       ```
2. Adjust the visualization options (e.g., graph, gauge, or table) to match your requirements.

---

#### **4. Save the Dashboard**
1. Click **Apply** to save the panel.
2. Name your dashboard (e.g., **GitLab Metrics Dashboard**) and save it.

---

#### **5. Monitor GitLab Metrics**
- Your dashboard will now display live GitLab metrics.
- Use the **Refresh Interval** option to set how often the dashboard updates.
- Add additional panels for more metrics as needed.

---

## **Summary**

1. Enable Prometheus in GitLab by editing the `gitlab.rb` file.
2. Reconfigure GitLab and test the `/metrics` endpoint.
3. Add GitLab to Prometheus' scrape configuration.
4. Integrate Prometheus with Grafana by adding it as a data source.
5. Create a custom dashboard in Grafana and link it to GitLab metrics using PromQL queries.

This setup provides a robust visualization of GitLab’s performance and health metrics. Let me know if you need specific PromQL examples or additional help!
