---
title: "Centralized Logging and Management: Deploying Elasticsearch, Kibana, and Fluentd on GCP"
seoTitle: "Centralized Logging with EFK on GCP"
seoDescription: "Deploy Elasticsearch, Kibana, and Fluentd on GCP for centralized log management with X-Pack security for secure access and real-time monitoring"
datePublished: Mon Nov 25 2024 14:49:15 GMT+0000 (Coordinated Universal Time)
cuid: cm3x58xqp000h09jsa2gm15v3
slug: centralized-logging-and-management-deploying-elasticsearch-kibana-and-fluentd-on-gcp
tags: monitoring, logging, fluentd, fluentbit, monitoring-and-logging, efk

---

---

**Introduction**

Effective log management is critical for monitoring, troubleshooting, and analyzing applications in real-time. In this post, we’re beginning a series on logging and management by setting up a centralized logging solution using **Elasticsearch** and **Kibana** with **X-Pack security enabled**. This ensures controlled access with user authentication.

We’ll deploy **Elasticsearch** and **Kibana** as Docker containers on a Google Cloud Platform (GCP) VM instance. Once the centralized logging system is set up, we’ll configure **Fluentd** to ship **NGINX access logs** to Elasticsearch for centralized monitoring.

---

### **Part 1: Setting Up the Centralized Server**

#### **Step 1: Create a GCP VM Instance**

1. **Log in to your GCP Console** and navigate to **Compute Engine &gt; VM Instances**.
    
2. **Create a new instance** with the following specifications:
    
    * **Name**: `logging-server`
        
    * **Machine type**: `e2-medium` (or higher, depending on traffic)
        
    * **Boot disk**: Ubuntu 20.04 LTS
        
    * **Firewall**: Allow HTTP and HTTPS traffic
        
3. SSH into the instance once it’s up and running.
    

---

#### **Step 2: Install Docker on the VM**

Update the VM and install Docker:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y docker.io
sudo systemctl enable --now docker
```

---

#### **Step 3: Deploy Elasticsearch with X-Pack Security**

1. Create a directory for Elasticsearch data:
    
    ```bash
    mkdir -p ~/elasticsearch_data
    sudo chmod 777 ~/elasticsearch_data
    ```
    
2. Run the Elasticsearch container with X-Pack enabled:
    
    ```bash
    docker run -d --name elasticsearch \
      -e "discovery.type=single-node" \
      -e "xpack.security.enabled=true" \
      -e "ELASTIC_PASSWORD=elastic123" \
      -v ~/elasticsearch_data:/usr/share/elasticsearch/data \
      -p 9200:9200 -p 9300:9300 \
      docker.elastic.co/elasticsearch/elasticsearch:8.10.1
    ```
    
3. Verify Elasticsearch is running:
    
    ```bash
    curl -u elastic:elastic123 http://<VM_EXTERNAL_IP>:9200
    ```
    
    Replace `<VM_EXTERNAL_IP>` with your instance’s external IP address.
    
    You should see a JSON response confirming Elasticsearch is active.
    

---

#### **Step 4: Deploy Kibana**

1. Run the Kibana container:
    
    ```bash
    docker run -d --name kibana \
      -e "ELASTICSEARCH_HOSTS=http://<VM_EXTERNAL_IP>:9200" \
      -e "ELASTICSEARCH_USERNAME=elastic" \
      -e "ELASTICSEARCH_PASSWORD=elastic123" \
      -p 5601:5601 \
      docker.elastic.co/kibana/kibana:8.10.1
    ```
    
2. Access Kibana in your browser at `http://<VM_EXTERNAL_IP>:5601`.  
    Use the **elastic** username and **elastic123** password to log in.
    

---

### **Part 2: Shipping Logs with Fluentd**

#### **Step 1: Install Fluentd**

On the server running NGINX (can be the same VM or another server):

1. Add the Fluentd repository:
    
    ```bash
    curl -fsSL https://packages.treasuredata.com/GPG-KEY-td-agent | sudo apt-key add -
    echo "deb http://packages.treasuredata.com/4/ubuntu/focal/ focal contrib" | sudo tee /etc/apt/sources.list.d/td-agent.list
    sudo apt update
    ```
    
2. Install Fluentd:
    
    ```bash
    sudo apt install -y td-agent
    ```
    

---

#### **Step 2: Configure Fluentd for NGINX Logs**

1. Install the Elasticsearch plugin for Fluentd:
    
    ```bash
    sudo /usr/sbin/td-agent-gem install fluent-plugin-elasticsearch
    ```
    
2. Update Fluentd’s configuration to ship logs to Elasticsearch. Open the configuration file:
    
    ```bash
    sudo nano /etc/td-agent/td-agent.conf
    ```
    
    Add the following configuration:
    
    ```yaml
    <source>
      @type tail
      path /var/log/nginx/access.log
      pos_file /var/log/td-agent/nginx-access.log.pos
      tag nginx.access
      format nginx
    </source>
    
    <match nginx.access>
      @type elasticsearch
      host <VM_EXTERNAL_IP>
      port 9200
      scheme http
      logstash_format true
      user elastic
      password elastic123
    </match>
    ```
    
3. Restart Fluentd:
    
    ```bash
    sudo systemctl restart td-agent
    ```
    

---

#### **Step 3: Verify Logs in Kibana**

1. Generate some logs by accessing your NGINX server:
    
    ```bash
    curl http://<NGINX_SERVER_IP>
    ```
    
2. In Kibana, go to **Discover** and create an index pattern for `logstash-*`.  
    You should see the NGINX logs appearing in real-time.
    
3. Create a **Data View** to analyze the index better and run queries.
    

---

### **Best Practices**

1. **Secure Access**: Ensure your Elasticsearch and Kibana instances are behind a firewall or protected using IP whitelisting.
    
2. **Persistent Data**: Use persistent disks for Elasticsearch data to prevent data loss on container restarts.
    
3. **Resource Management**: Monitor resource usage and scale the VM as needed to handle increased log volumes.
    
4. **Log Rotation**: Set up log rotation on the Fluentd source (e.g., NGINX logs) to prevent disk exhaustion.
    

---

### **Conclusion**

This setup provides a robust centralized logging system with Elasticsearch and Kibana, secured by X-Pack authentication. By using Fluentd to ship logs, you can monitor and analyze NGINX logs or any other application logs from a single interface.

In the next post, we’ll expand this setup to handle high-volume production traffic and discuss advanced index management techniques. Stay tuned!

Have questions or insights? Share them in the comments below or reach out on LinkedIn!