# Online Threat Feed Creator for FortiGate

A Python-based tool for creating and managing custom threat intelligence feeds for FortiGate firewalls. This tool aggregates threat data from multiple online sources and generates CSV-formatted feeds that can be hosted on a web server for FortiGate ingestion.

**Status**: ✅ Working / Production-Ready

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [What's Included](#whats-included)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Web Server Setup](#web-server-setup)
- [FortiGate Integration](#fortigate-integration)
- [Supported Threat Sources](#supported-threat-sources)
- [Advanced Configuration](#advanced-configuration)
- [Docker Deployment](#docker-deployment)
- [Troubleshooting](#troubleshooting)
- [Security Considerations](#security-considerations)
- [Contributing](#contributing)
- [License](#license)

## Overview

This tool automates the process of collecting threat intelligence from various online sources and formatting it into feeds that FortiGate firewalls can consume. Unlike manual feed creation, this tool handles data aggregation, deduplication, and formatting automatically on a configurable schedule.

The generated feeds must be hosted on an accessible web server (nginx, Apache, etc.) since FortiGate retrieves them via HTTP/HTTPS URLs.

## Features

- **Automated Feed Generation**: Regularly fetches and processes threat data on a defined schedule
- **Multiple Threat Sources**: Aggregates data from publicly available threat intelligence feeds
- **CSV Export Format**: Generates feeds in FortiGate-compatible CSV format
- **IP & Domain Support**: Creates separate feeds for IP addresses and domain names
- **Configurable Output**: Control feed size, update intervals, and output formats
- **Environment-Based Configuration**: Secure API key and credential management
- **Logging**: Comprehensive logging for monitoring and debugging
- **Flexible Filtering**: Exclude private IPs, filter by confidence levels, and more

## What's Included

This repository contains the following components:

- **threat_feed_creator.py**: Main script for fetching and aggregating threat data
- **config.yaml / config.example.yaml**: Configuration template for customizing behavior
- **requirements.txt**: Python dependencies (requests, pyyaml, etc.)
- **Dockerfile & docker-compose.yml**: Optional containerized deployment files
- **output/ directory**: Location where generated feed files are stored
- **LICENSE**: MIT License file

The tool currently supports threat sources including AlienVault OTX, Abuse.ch, Blocklist.de, OpenPhish, and URLhaus.

## Prerequisites

- **Python 3.8** or higher
- **pip** package manager
- **A web server** (nginx, Apache, or similar) to host the generated feeds
- **FortiGate firewall** with external threat feed support enabled
- **Internet connectivity** to reach threat intelligence providers

## Installation

### 1. Clone the Repository

```bash
git clone https://github.com/mobcer/online-Threat-feed-creator-for-Fortigate.git
cd online-Threat-feed-creator-for-Fortigate
```

### 2. Install Python Dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure the Tool

```bash
cp config.example.yaml config.yaml
```

Edit `config.yaml` with your settings (see [Configuration](#configuration) section).

## Configuration

### Basic Configuration (config.yaml)

```yaml
# Threat sources to enable
sources:
  - name: "alienvault_otx"
    enabled: true
    api_key: "your_api_key_here"
  
  - name: "abuse_ch"
    enabled: true
    feed_type: "ips"
  
  - name: "blocklist_de"
    enabled: true
  
  - name: "openphish"
    enabled: true
  
  - name: "urlhaus"
    enabled: true

# Output configuration
output:
  format: "csv"
  update_interval: 3600        # seconds (1 hour)
  max_entries: 10000
  output_directory: "./output"

# FortiGate feed naming
fortigate:
  feed_name: "custom_threat_feed"
  category: "security"
```

### Environment Variables

Store sensitive API keys as environment variables for security:

```bash
export OTX_API_KEY="your_alienvault_key"
export ABUSE_CH_API_KEY="your_abuse_ch_key"
```

### Advanced Filtering

```yaml
filters:
  exclude_private_ips: true
  min_confidence: 70
  allowed_countries: ["US", "CA", "GB"]
  exclude_ips:
    - "192.168.1.1"
    - "10.0.0.0/8"

logging:
  level: "INFO"
  file: "/var/log/threat_feed.log"
  max_size: "10MB"
  backup_count: 5
```

## Usage

### Run the Threat Feed Creator

```bash
python threat_feed_creator.py
```

The script will:
1. Fetch threat data from configured sources
2. Aggregate and deduplicate the data
3. Apply configured filters
4. Generate CSV files in the `output/` directory

### Output Files

Generated feeds will be created in the output directory:
- `ip_threats.csv` - Malicious IP addresses
- `domain_threats.csv` - Malicious domains
- `url_threats.csv` - Malicious URLs

Each file contains threat data in FortiGate-compatible CSV format.

## Web Server Setup

FortiGate cannot directly read files from your machine. You must host the generated feeds on a web server accessible to your FortiGate firewall.

### Nginx Setup Example

```bash
# Install nginx (Ubuntu/Debian)
sudo apt-get install nginx

# Create directory for threat feeds
sudo mkdir -p /var/www/threat_feeds
sudo chown $USER:$USER /var/www/threat_feeds

# Copy generated feeds to web server directory
cp output/*.csv /var/www/threat_feeds/
```

Create nginx configuration (`/etc/nginx/sites-available/threat-feeds`):

```nginx
server {
    listen 80;
    server_name your-server.com;

    location /feeds/ {
        alias /var/www/threat_feeds/;
        autoindex off;
    }

    # Optional: Redirect to HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-server.com;

    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;

    location /feeds/ {
        alias /var/www/threat_feeds/;
        autoindex off;
    }
}
```

Enable the configuration:

```bash
sudo ln -s /etc/nginx/sites-available/threat-feeds /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### Automation with Cron

Schedule automatic feed updates every hour:

```bash
crontab -e
```

Add:

```
0 * * * * cd /path/to/online-Threat-feed-creator-for-Fortigate && python threat_feed_creator.py && cp output/*.csv /var/www/threat_feeds/
```

Then reload nginx or use a reload script to pick up changes:

```bash
0 * * * * sudo systemctl reload nginx
```

## FortiGate Integration

### Web Interface Setup

1. Log in to your FortiGate firewall
2. Navigate to **Security Fabric > External Threat Feeds**
3. Click **Create New** to add a new threat feed
4. Configure the following:
   - **Name**: `Custom_Threat_Feed_IPs`
   - **Type**: IPv4 Address
   - **Source**: `https://your-server.com/feeds/ip_threats.csv`
   - **Update Frequency**: Every 5 minutes (or as desired)
5. Repeat for domain feeds:
   - **Name**: `Custom_Threat_Feed_Domains`
   - **Type**: Domain
   - **Source**: `https://your-server.com/feeds/domain_threats.csv`

### CLI Configuration

Alternatively, use FortiGate CLI:

```
config system external-resource
  edit "Custom_Threat_Feed_IPs"
    set type address
    set resource "https://your-server.com/feeds/ip_threats.csv"
    set refresh-rate 300
  next
  edit "Custom_Threat_Feed_Domains"
    set type domain
    set resource "https://your-server.com/feeds/domain_threats.csv"
    set refresh-rate 300
  next
end
```

## Supported Threat Sources

| Source | Type | API Required | Notes |
|--------|------|--------------|-------|
| AlienVault OTX | IPs, Domains | Yes (Free) | Comprehensive threat intelligence database |
| Abuse.ch | IPs, URLs | No | Malware and phishing data |
| Blocklist.de | IPs | No | Failed login attempts and exploit data |
| OpenPhish | URLs | No | Active phishing detection |
| URLhaus | URLs, Domains | No | Malware distribution tracking |

## Advanced Configuration

### Custom Threat Sources

To add additional threat sources, extend the threat feed creator:

```python
# Add custom source logic to threat_feed_creator.py
class CustomThreatSource:
    def __init__(self, config):
        self.config = config
    
    def fetch_threats(self):
        # Your custom logic to fetch threats
        threats = []
        # Process and return threat data
        return threats
```

### Performance Tuning

For large deployments, optimize performance:

```yaml
output:
  max_entries: 50000          # Increase for more threats
  batch_size: 5000            # Process in batches
  update_interval: 7200       # Longer intervals for stability
  cleanup_old_feeds: true
  keep_backups: 3
```

## Docker Deployment

### Using Docker Compose (Recommended)

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  threat-feed:
    build: .
    volumes:
      - ./config.yaml:/app/config.yaml
      - ./output:/app/output
    environment:
      - OTX_API_KEY=${OTX_API_KEY}
      - ABUSE_CH_API_KEY=${ABUSE_CH_API_KEY}
    restart: unless-stopped
    deploy:
      restart_policy:
        condition: on-failure
        delay: 30s
```

Run with Docker Compose:

```bash
docker-compose up -d
```

### Manual Docker Build

```bash
docker build -t fortigate-threat-feed .
docker run -d \
  -v $(pwd)/config.yaml:/app/config.yaml \
  -v $(pwd)/output:/app/output \
  -e OTX_API_KEY="your_key" \
  --restart unless-stopped \
  fortigate-threat-feed
```

## Troubleshooting

### Issue: "Connection refused" when FortiGate tries to fetch feeds

- **Solution**: Verify your web server is running and accessible from the FortiGate network
- Check firewall rules allow traffic on port 80/443
- Test with `curl https://your-server.com/feeds/ip_threats.csv` from FortiGate CLI

### Issue: Empty threat feeds generated

- **Solution**: Check that threat sources are enabled in `config.yaml`
- Verify API keys are set correctly for sources that require authentication
- Check logs for connection errors to threat intelligence providers

### Issue: FortiGate not updating feeds

- **Solution**: Verify the feed URL is correct and accessible
- Check FortiGate logs: System > Event Log > External Threats
- Ensure DNS resolution works on the FortiGate
- Verify feed format matches FortiGate requirements (CSV with proper headers)

### Issue: High memory usage or slow script execution

- **Solution**: Reduce `max_entries` in configuration
- Increase `update_interval` to run less frequently
- Process feeds in smaller batches

### Issue: "Permission denied" when accessing output directory

- **Solution**: Ensure the script has write permissions:
```bash
chmod -R 755 output/
```

## Security Considerations

- **API Keys**: Never commit API keys to version control. Use environment variables or `.env` files
- **HTTPS**: Always serve threat feeds over HTTPS with valid SSL certificates
- **Access Control**: Restrict access to your threat feed server (firewall rules, IP whitelisting)
- **Rate Limiting**: Respect API rate limits of threat intelligence providers to avoid blocks
- **Log Monitoring**: Regularly review logs for suspicious activity or errors
- **Feed Validation**: Verify feed integrity before deploying to production
- **Backup Feeds**: Keep backups of previous feed versions in case of corruption

## Contributing

Contributions are welcome! To contribute:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m 'Add your feature'`
4. Push to the branch: `git push origin feature/your-feature`
5. Open a Pull Request

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

---

**Note**: Always test threat feeds in a controlled environment before deploying to production FortiGate firewalls.

**Questions?** Check the [issue tracker](../../issues) or create a new discussion.
