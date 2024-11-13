The elasticsearch.yml file is the main configuration file for Elasticsearch, defining settings that control the behavior, performance, security, and networking of the Elasticsearch instance. Properly planning and configuring this file is essential for optimizing your Elasticsearch deployment, ensuring it meets your requirements in areas such as performance, scalability, and security.

Key Sections in elasticsearch.yml

1. Cluster and Node Settings

cluster.name: Sets the name of the cluster, which is critical when running multiple clusters. For example:

cluster.name: my_elasticsearch_cluster

node.name: Assigns a name to each node, which can help identify nodes in logs and the Kibana monitoring interface.

node.name: node-1

node.roles: Defines the roles assigned to the node, such as master, data, or ingest. For example:

node.roles: ["master", "data"]



2. Network Settings

network.host: Specifies the IP address or hostname where the node will listen for requests. Common settings are localhost (for single-node development) or a specific IP address for a cluster.

network.host: 0.0.0.0

http.port: Sets the HTTP port for Elasticsearch. By default, it's 9200, but this can be changed if there are conflicts.

http.port: 9200

transport.port: Used for communication between nodes within the cluster. This defaults to 9300.



3. Discovery and Cluster Formation Settings

discovery.seed_hosts: Lists the initial nodes that the cluster will use to discover other nodes. Required for multi-node clusters.

discovery.seed_hosts: ["node1.example.com", "node2.example.com"]

cluster.initial_master_nodes: Specifies the initial set of master-eligible nodes for forming a new cluster. This setting prevents split-brain issues in clusters with multiple master-eligible nodes.

cluster.initial_master_nodes: ["node1", "node2"]



4. Path Settings

path.data: Specifies the location on disk where Elasticsearch will store data, like indexes and translogs.

path.data: /var/lib/elasticsearch/data

path.logs: Location for Elasticsearch logs. Useful to set this to a high-capacity disk in production.

path.logs: /var/log/elasticsearch



5. Memory and Performance Settings

bootstrap.memory_lock: Prevents the operating system from swapping Elasticsearch memory to disk, which can improve performance and is recommended for production.

bootstrap.memory_lock: true

indices.fielddata.cache.size: Sets the maximum percentage of heap memory used for field data, helping optimize memory usage when performing aggregations.



6. Security Settings (if security is enabled)

xpack.security.enabled: Enables built-in security features like authentication and authorization.

xpack.security.enabled: true

xpack.security.transport.ssl.enabled: Enables SSL/TLS for communication between nodes, which is essential for secure clusters.

xpack.security.transport.ssl.enabled: true



7. Index Settings

index.number_of_shards and index.number_of_replicas: Configure the default number of shards and replicas for new indexes, impacting performance and fault tolerance.

index.number_of_shards: 5
index.number_of_replicas: 1




Planning elasticsearch.yml

Step 1: Define Your Cluster Topology

Single-node setup: Set basic networking options (network.host: localhost) without any clustering configurations.

Multi-node setup: Decide how many master, data, and ingest nodes you’ll need based on your workload.

Plan for specific nodes with dedicated roles (e.g., master, data) for better performance and stability in production.


Step 2: Set Up Networking and Discovery

If your cluster spans multiple nodes, configure network.host and discovery.seed_hosts to facilitate node communication.

Define cluster.initial_master_nodes for multi-master-eligible nodes to prevent split-brain situations.


Step 3: Configure Memory and Storage

Allocate storage paths (path.data and path.logs) on high-performance disks, ideally separate from the operating system disk.

Enable bootstrap.memory_lock to prevent swapping, and set Java heap size (outside elasticsearch.yml) according to available system memory.


Step 4: Plan for Security (if required)

Enable xpack.security.enabled and SSL/TLS settings for a secure setup, especially if your cluster is accessible outside the local network.

Define user authentication and permissions to control access to different parts of the cluster and data.


Step 5: Set Default Indexing Parameters

Adjust default settings for index.number_of_shards and index.number_of_replicas based on data size and availability needs. Fewer shards are typically better for smaller clusters, while more shards may benefit larger clusters with high data volumes.


Step 6: Test and Monitor

Once the elasticsearch.yml configuration is set up, deploy it in a test environment to ensure everything works as expected.

Use monitoring tools like Kibana and Elasticsearch’s APIs to keep track of performance, disk usage, and error logs, making adjustments to elasticsearch.yml as needed.


Example elasticsearch.yml Configuration

cluster.name: my_elasticsearch_cluster
node.name: node-1
node.roles: ["master", "data"]

path.data: /var/lib/elasticsearch/data
path.logs: /var/log/elasticsearch

network.host: 0.0.0.0
http.port: 9200
transport.port: 9300

discovery.seed_hosts: ["node1.example.com", "node2.example.com"]
cluster.initial_master_nodes: ["node1", "node2"]

bootstrap.memory_lock: true

xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true

Additional Resources

To stay updated with best practices and ensure compatibility, consult:

The official Elasticsearch Reference for documentation.

Elasticsearch blogs and forums for community-driven insights and configurations optimized for specific use cases.


This setup ensures your Elasticsearch instance is optimized for your environment, with the correct balance of performance, security, and resource management.

