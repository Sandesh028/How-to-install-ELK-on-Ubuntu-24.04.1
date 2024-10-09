# How-to-install-ELK-on-Ubuntu-24.04.1
Here is a detailed `README.md` file that you can use for your GitHub repository, describing the steps to install and configure the ELK Stack (Elasticsearch, Logstash, Kibana) along with Zeek for monitoring and log analysis:

# ELK Stack + Zeek Setup on Ubuntu for Monitoring and Log Analysis

This guide provides detailed instructions on installing and configuring the ELK (Elasticsearch, Logstash, Kibana) Stack along with Zeek on Ubuntu. It is tailored for those who want to monitor system logs, browser logs, and network activity across different users in a virtual environment or real-world systems.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1: Install ELK Stack](#step-1-install-elk-stack)
- [Step 2: Install Zeek](#step-2-install-zeek)
- [Step 3: Enable Elasticsearch Security Features](#step-3-enable-elasticsearch-security-features)
- [Step 4: Install and Configure Logstash](#step-4-install-and-configure-logstash)
- [Step 5: Setup Kibana and Enable Logs Visualization](#step-5-setup-kibana-and-enable-logs-visualization)
- [Step 6: Install and Enroll Elastic Agent with Fleet](#step-6-install-and-enroll-elastic-agent-with-fleet)
- [Step 7: Configure System to Automatically Start ELK and Zeek on Boot](#step-7-configure-system-to-automatically-start-elk-and-zeek-on-boot)
- [Step 8: Collect and Visualize Logs](#step-8-collect-and-visualize-logs)
- [Troubleshooting](#troubleshooting)

## Prerequisites

- Ubuntu 20.04+ or newer installed (VirtualBox or native machine)
- Internet access
- Sudo privileges

## Step 1: Install ELK Stack

1. **Add Elasticsearch Repository**:
   ```bash
   sudo apt update && sudo apt install apt-transport-https curl
   curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
   sudo sh -c 'echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" > /etc/apt/sources.list.d/elastic-7.x.list'
   ```

2. **Install Elasticsearch, Logstash, and Kibana**:
   ```bash
   sudo apt update && sudo apt install elasticsearch logstash kibana
   ```

3. **Enable and Start Elasticsearch, Logstash, and Kibana**:
   ```bash
   sudo systemctl enable elasticsearch
   sudo systemctl start elasticsearch

   sudo systemctl enable logstash
   sudo systemctl start logstash

   sudo systemctl enable kibana
   sudo systemctl start kibana
   ```

4. **Access Kibana**: 
   Open your browser and go to `http://localhost:5601`.

## Step 2: Install Zeek

1. **Add Zeek Repository**:
   ```bash
   sudo apt install curl gnupg
   curl -s https://download.opensuse.org/repositories/security:zeek/xUbuntu_$(lsb_release -rs)/Release.key | sudo apt-key add -
   sudo sh -c "echo 'deb http://download.opensuse.org/repositories/security:/zeek/xUbuntu_$(lsb_release -rs)/ /' > /etc/apt/sources.list.d/security:zeek.list"
   ```

2. **Install Zeek**:
   ```bash
   sudo apt update && sudo apt install zeek
   ```

3. **Start Zeek**:
   ```bash
   sudo /opt/zeek/bin/zeekctl deploy
   ```

## Step 3: Enable Elasticsearch Security Features

1. Open the Elasticsearch configuration file:
   ```bash
   sudo nano /etc/elasticsearch/elasticsearch.yml
   ```

2. Add the following lines to enable security features:
   ```yaml
   xpack.security.enabled: true
   xpack.security.authc.api_key.enabled: true
   ```

3. Restart Elasticsearch:
   ```bash
   sudo systemctl restart elasticsearch
   ```

## Step 4: Install and Configure Logstash

1. Create a Logstash configuration file for Zeek:
   ```bash
   sudo nano /etc/logstash/conf.d/zeek.conf
   ```

2. Add the following content to process Zeek logs:
   ```plaintext
   input {
     file {
       path => "/opt/zeek/logs/current/*.log"
       start_position => "beginning"
     }
   }
   filter {
     # Add filtering here if needed
   }
   output {
     elasticsearch {
       hosts => ["localhost:9200"]
     }
   }
   ```

3. Start Logstash:
   ```bash
   sudo systemctl start logstash
   ```

## Step 5: Setup Kibana and Enable Logs Visualization

1. **Enable Logs Visualization in Kibana**:
   - Access Kibana at `http://localhost:5601`.
   - Go to **Observability** > **Logs** and set up data streams to start visualizing logs.

## Step 6: Install and Enroll Elastic Agent with Fleet

1. **Download Elastic Agent**:
   Go to [Elastic's download page](https://www.elastic.co/downloads/elastic-agent) and download the Elastic Agent binary for Linux.

2. **Install Elastic Agent**:
   ```bash
   sudo ./elastic-agent install
   ```

3. **Enroll Agent in Fleet**:
   - Use Fleet server URL and the service token generated from Kibana to enroll the agent:
     ```bash
     sudo ./elastic-agent install --fleet-server-es=http://localhost:9200 --fleet-server-service-token=YOUR_TOKEN --fleet-server-insecure-http
     ```

## Step 7: Configure System to Automatically Start ELK and Zeek on Boot

1. **Enable Elasticsearch, Logstash, Kibana on Boot**:
   ```bash
   sudo systemctl enable elasticsearch
   sudo systemctl enable logstash
   sudo systemctl enable kibana
   ```

2. **Create a systemd service for Zeek**:
   ```bash
   sudo nano /etc/systemd/system/zeek.service
   ```
   Add the following:
   ```ini
   [Unit]
   Description=Zeek Network Security Monitor
   After=network.target

   [Service]
   Type=simple
   ExecStart=/opt/zeek/bin/zeekctl deploy
   ExecStop=/opt/zeek/bin/zeekctl stop
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   ```

3. **Enable Zeek on Boot**:
   ```bash
   sudo systemctl enable zeek
   ```

## Step 8: Collect and Visualize Logs

1. **System Logs**: Use Filebeat to collect system logs by enabling system module:
   ```bash
   sudo filebeat modules enable system
   sudo filebeat setup
   sudo systemctl start filebeat
   ```

2. **Zeek Logs**: The Zeek logs will be processed by Logstash and visualized in Kibana.

3. **Access Logs**: In Kibana, go to **Observability** > **Logs** to see and analyze the collected logs.

## Troubleshooting

- If Elasticsearch or Kibana fails to start, check the logs using:
   ```bash
   sudo journalctl -u elasticsearch
   sudo journalctl -u kibana
   ```

- For Elastic Agent issues, check its status:
   ```bash
   sudo systemctl status elastic-agent
   ```


