Online Threat Feed Creator for FortiGate

A powerful tool for creating and managing custom threat intelligence feeds for FortiGate firewalls. This tool aggregates threat data from multiple online sources and formats it for seamless integration with FortiGate's threat feed functionality.
🌟 Features

    Multi-Source Threat Intelligence: Aggregates data from various threat intelligence sources

    FortiGate-Compatible Output: Generates feeds in formats supported by FortiGate firewalls

    Automated Updates: Scheduled fetching and processing of threat data

    Customizable Filtering: Configurable rules for including/excluding specific threats

    IP & Domain Feeds: Supports both IP address and domain name threat feeds

    Easy Deployment: Simple setup with Docker and configuration files

🚀 Quick Start
Prerequisites

    Python 3.8 or higher

    FortiGate firewall with external threat feed support

    (Optional) Docker for containerized deployment

Installation

    Clone the repository
    bash

git clone https://github.com/mobcer/online-Threat-feed-creator-for-Fortigate.git
cd online-Threat-feed-creator-for-Fortigate

Install dependencies
bash

pip install -r requirements.txt

Configure the tool
bash

cp config.example.yaml config.yaml
# Edit config.yaml with your preferred settings

Basic Usage

    Run the threat feed creator
    bash

python threat_feed_creator.py

    The tool will generate threat feeds in the output/ directory

    Configure FortiGate to use the generated feeds

        Upload the generated files to a web server accessible by your FortiGate

        Configure FortiGate external threat feeds to point to these URLs

⚙️ Configuration
Configuration File (config.yaml)
yaml

sources:
  - name: "alienvault_otx"
    enabled: true
    api_key: "your_api_key_here"
    
  - name: "abuse_ch"
    enabled: true
    feed_type: "ips"
    
  - name: "blocklist_de"
    enabled: true

output:
  format: "csv"  # or "txt", "json"
  update_interval: 3600  # seconds
  max_entries: 10000
  
fortigate:
  feed_name: "custom_threat_feed"
  category: "security"

Environment Variables
bash

export OTX_API_KEY="your_alienvault_key"
export ABUSE_CH_API_KEY="your_abuse_ch_key"

🐳 Docker Deployment
Using Docker Compose (Recommended)

    Create docker-compose.yml
    yaml

version: '3.8'
services:
  threat-feed:
    build: .
    volumes:
      - ./config.yaml:/app/config.yaml
      - ./output:/app/output
    environment:
      - OTX_API_KEY=your_api_key
    restart: unless-stopped

Run with Docker Compose
bash

docker-compose up -d

Manual Docker Build
bash

docker build -t fortigate-threat-feed .
docker run -d \
  -v $(pwd)/output:/app/output \
  -e OTX_API_KEY="your_key" \
  fortigate-threat-feed

🔧 FortiGate Configuration
Web Interface Setup

    Navigate to Security Fabric > External Threat Feeds

    Click "Create New"

    Configure feed settings:

        Name: Custom_Threat_Feed

        Type: IPv4 Address/URL/Domain

        Source: URL of your generated feed

        Update Frequency: Every 5 minutes

CLI Configuration
bash

config system external-resource
    edit "Custom_Threat_Feed_IP"
        set type address
        set resource "https://your-server.com/feeds/ip_threats.csv"
        set refresh-rate 300
    next
    edit "Custom_Threat_Feed_Domain"
        set type domain
        set resource "https://your-server.com/feeds/domain_threats.csv"
        set refresh-rate 300
    next
end

📊 Supported Threat Intelligence Sources
Source	Type	API Required	Notes
AlienVault OTX	IPs, Domains	Yes (Free)	Comprehensive threat data
Abuse.ch	IPs, URLs	No	Malware and phishing
Blocklist.de	IPs	No	Failed login attempts
OpenPhish	URLs	No	Active phishing sites
URLhaus	URLs, Domains	No	Malware distribution
🔒 Security Considerations

    API Keys: Store API keys securely using environment variables

    HTTPS: Serve threat feeds over HTTPS

    Access Control: Restrict access to your threat feed server

    Rate Limiting: Respect API rate limits of threat intelligence providers

📈 Monitoring and Logging
Enable Logging
yaml

logging:
  level: "INFO"
  file: "/var/log/threat_feed.log"
  max_size: "10MB"
  backup_count: 5

Health Check Endpoint

The tool includes a health check endpoint at /health when running as a web service.
🛠️ Advanced Usage
Custom Threat Sources

Add custom threat sources by extending the ThreatSource class:
python

from threat_feed_creator.sources import ThreatSource

class CustomThreatSource(ThreatSource):
    def fetch_threats(self):
        # Your custom logic here
        return threats_list

Filtering Rules
yaml

filters:
  exclude_private_ips: true
  min_confidence: 70
  allowed_countries: ["US", "CA", "GB"]
  exclude_ips: ["192.168.1.1"]

🤝 Contributing

We welcome contributions! Please see our Contributing Guide for details.

    Fork the repository

    Create a feature branch (git checkout -b feature/amazing-feature)

    Commit your changes (git commit -m 'Add amazing feature')

    Push to the branch (git push origin feature/amazing-feature)

    Open a Pull Request

📝 License

This project is licensed under the MIT License - see the LICENSE file for details.
🆘 Support

    📖 Documentation

    🐛 Issue Tracker

    💬 Discussions

🙏 Acknowledgments

    Threat intelligence providers for their valuable data

    Fortinet for the FortiGate platform

    Contributors and users of this project

Note: Always test threat feeds in a controlled environment before deploying to production.
