# Ollama Installation Guide

Ollama is a free and open-source tool for running large language models (LLMs) locally on your machine. It serves as a FOSS alternative to cloud-based AI services like OpenAI API, Anthropic Claude API, Google's Gemini API, or Azure OpenAI Service. Ollama enables privacy-focused AI deployment, offline inference, and cost-effective local AI processing with support for popular models like Llama 3, Code Llama, Mistral, and many others.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

### Hardware Requirements
- **CPU**: Modern 64-bit processor (x86_64 or ARM64)
- **RAM**: 8GB minimum (16GB+ recommended for larger models)
- **Storage**: 10GB+ free space for models
- **GPU**: Optional but recommended (NVIDIA with CUDA, AMD ROCm, or Apple Metal)

### Software Requirements
- **Operating System**: Linux, macOS, or Windows
- **Internet**: Required for initial model downloads
- **Docker**: Optional for containerized deployment

### Network Requirements
- **Ports**: 
  - 11434: Default API server port
- **Bandwidth**: High-speed internet for model downloads (models range from 1GB to 70GB+)

## 2. Supported Operating Systems

Ollama officially supports:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04 LTS / 22.04 LTS / 24.04 LTS
- Arch Linux
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- macOS 11+ (Big Sur and later)
- Windows 10/11

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Method 1: Official installer script
curl -fsSL https://ollama.com/install.sh | sh

# Method 2: Manual installation
# Download latest release
curl -L https://ollama.com/download/linux-amd64 -o /usr/local/bin/ollama
chmod +x /usr/local/bin/ollama

# Create ollama user
sudo useradd -r -s /bin/false -m -d /usr/share/ollama ollama
sudo usermod -a -G render,video ollama

# Create systemd service
sudo tee /etc/systemd/system/ollama.service > /dev/null << 'EOF'
[Unit]
Description=Ollama Service
After=network-online.target

[Service]
ExecStart=/usr/local/bin/ollama serve
User=ollama
Group=ollama
Restart=always
RestartSec=3
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Environment="OLLAMA_HOST=0.0.0.0"

[Install]
WantedBy=default.target
EOF

# Enable and start service
sudo systemctl daemon-reload
sudo systemctl enable --now ollama
```

### Debian/Ubuntu

```bash
# Method 1: Official installer script
curl -fsSL https://ollama.com/install.sh | sh

# Method 2: Package installation (if available)
# Add official repository
curl -fsSL https://ollama.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/ollama-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/ollama-keyring.gpg] https://ollama.com/debian stable main" | sudo tee /etc/apt/sources.list.d/ollama.list

# Install package
sudo apt update
sudo apt install -y ollama

# Start service
sudo systemctl enable --now ollama
```

### Arch Linux

```bash
# Install from AUR
yay -S ollama-bin
# or
paru -S ollama-bin

# Alternative: Build from source
yay -S ollama

# Enable and start service
sudo systemctl enable --now ollama
```

### Alpine Linux

```bash
# Install dependencies
apk add --no-cache curl

# Install Ollama binary
curl -L https://ollama.com/download/linux-amd64 -o /usr/local/bin/ollama
chmod +x /usr/local/bin/ollama

# Create ollama user
adduser -D -s /bin/false -h /usr/share/ollama ollama
addgroup ollama video
addgroup ollama render

# Create OpenRC service
tee /etc/init.d/ollama > /dev/null << 'EOF'
#!/sbin/openrc-run

description="Ollama Service"
command="/usr/local/bin/ollama"
command_args="serve"
command_user="ollama"
command_group="ollama"
pidfile="/run/ollama.pid"
command_background="yes"

depend() {
    need net
    after firewall
}

start_pre() {
    export OLLAMA_HOST="0.0.0.0"
    checkpath --directory --owner ollama:ollama --mode 0755 /run/ollama
}
EOF

chmod +x /etc/init.d/ollama
rc-update add ollama default
rc-service ollama start
```

### openSUSE

```bash
# Install via zypper (if available) or manual installation
sudo zypper refresh

# Manual installation
curl -L https://ollama.com/download/linux-amd64 -o /usr/local/bin/ollama
chmod +x /usr/local/bin/ollama

# Create ollama user
sudo useradd -r -s /bin/false -m -d /usr/share/ollama ollama
sudo usermod -a -G video,render ollama

# Create systemd service (same as RHEL)
sudo systemctl enable --now ollama
```

### macOS

```bash
# Method 1: Official app installer
# Download from https://ollama.com/download/mac

# Method 2: Homebrew
brew install ollama

# Method 3: Manual installation
curl -L https://ollama.com/download/darwin-amd64 -o /usr/local/bin/ollama
chmod +x /usr/local/bin/ollama

# Start Ollama
ollama serve &
```

### Windows

```powershell
# Method 1: Official installer
# Download and run installer from https://ollama.com/download/windows

# Method 2: Winget
winget install Ollama.Ollama

# Method 3: Chocolatey
choco install ollama

# Method 4: Scoop
scoop bucket add extras
scoop install ollama

# Start Ollama service (automatic with installer)
```

## 4. Configuration

### Environment Variables

Create `/etc/systemd/system/ollama.service.d/override.conf`:
```ini
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
Environment="OLLAMA_MODELS=/var/lib/ollama/models"
Environment="OLLAMA_NUM_PARALLEL=2"
Environment="OLLAMA_MAX_LOADED_MODELS=3"
Environment="OLLAMA_FLASH_ATTENTION=1"
```

### Configuration Options

```bash
# Set custom models directory
export OLLAMA_MODELS=/custom/path/to/models

# Configure host and port
export OLLAMA_HOST=127.0.0.1:11434

# GPU configuration
export CUDA_VISIBLE_DEVICES=0,1  # Use specific GPUs
export OLLAMA_GPU_OVERHEAD=0     # Reduce GPU memory overhead

# Performance tuning
export OLLAMA_NUM_PARALLEL=4     # Parallel requests
export OLLAMA_MAX_LOADED_MODELS=2 # Max models in memory
export OLLAMA_FLASH_ATTENTION=1  # Enable flash attention
```

### Model Management

```bash
# Download and run models
ollama pull llama3.1:8b
ollama pull codellama:13b
ollama pull mistral:7b

# List installed models
ollama list

# Run a model interactively
ollama run llama3.1:8b

# Remove a model
ollama rm llama3.1:8b

# Show model information
ollama show llama3.1:8b
```

## 5. Service Management

### systemd (Linux)

```bash
# Start/stop/restart service
sudo systemctl start ollama
sudo systemctl stop ollama
sudo systemctl restart ollama

# Check service status
sudo systemctl status ollama

# View logs
sudo journalctl -u ollama -f

# Enable/disable auto-start
sudo systemctl enable ollama
sudo systemctl disable ollama
```

### Manual Service Management

```bash
# Start Ollama server
ollama serve

# Start with custom configuration
OLLAMA_HOST=0.0.0.0:11434 ollama serve

# Background process
nohup ollama serve > /var/log/ollama.log 2>&1 &
```

### Windows Service Management

```powershell
# Check service status
Get-Service Ollama

# Start/stop service
Start-Service Ollama
Stop-Service Ollama

# Restart service
Restart-Service Ollama
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u ollama -n 50

# Check if port is in use
sudo netstat -tlnp | grep 11434

# Verify user permissions
sudo -u ollama /usr/local/bin/ollama serve
```

2. **GPU not detected**:
```bash
# Check NVIDIA GPU
nvidia-smi

# Check CUDA installation
nvcc --version

# Check Ollama GPU support
ollama info
```

3. **Model download fails**:
```bash
# Check internet connectivity
curl -I https://ollama.com

# Check disk space
df -h /var/lib/ollama

# Manual model download
curl -L https://huggingface.co/model-url -o model-file
```

4. **High memory usage**:
```bash
# Check model memory usage
ollama ps

# Reduce loaded models
export OLLAMA_MAX_LOADED_MODELS=1

# Monitor system resources
htop
```

### Debug Mode

```bash
# Enable debug logging
export OLLAMA_DEBUG=1
ollama serve

# Verbose API logging
export OLLAMA_VERBOSE=1
```

## 7. Security Considerations

### Network Security

```bash
# Bind to localhost only (default)
export OLLAMA_HOST=127.0.0.1:11434

# Configure firewall (if exposing externally)
sudo firewall-cmd --permanent --add-port=11434/tcp
sudo firewall-cmd --reload

# Use reverse proxy for external access
```

### Reverse Proxy Configuration (nginx)

```nginx
server {
    listen 80;
    server_name ollama.example.com;
    
    location / {
        proxy_pass http://127.0.0.1:11434;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Authentication Setup

```bash
# Ollama doesn't have built-in auth, use reverse proxy
# Example with basic auth in nginx:
sudo apt install apache2-utils
sudo htpasswd -c /etc/nginx/.htpasswd ollama_user

# Add to nginx config:
# auth_basic "Ollama Access";
# auth_basic_user_file /etc/nginx/.htpasswd;
```

### File Permissions

```bash
# Secure model directory
sudo chown -R ollama:ollama /var/lib/ollama
sudo chmod -R 750 /var/lib/ollama

# Secure configuration files
sudo chmod 640 /etc/systemd/system/ollama.service
sudo chown root:root /etc/systemd/system/ollama.service
```

## 8. Performance Tuning

### GPU Optimization

```bash
# NVIDIA GPU settings
export CUDA_VISIBLE_DEVICES=0,1
export OLLAMA_GPU_OVERHEAD=0

# Check GPU utilization
nvidia-smi -l 1

# AMD GPU (ROCm)
export HSA_OVERRIDE_GFX_VERSION=10.3.0
export ROCM_PATH=/opt/rocm
```

### CPU Optimization

```bash
# Set CPU affinity
taskset -c 0-7 ollama serve

# Adjust parallel processing
export OLLAMA_NUM_PARALLEL=4
export OLLAMA_MAX_LOADED_MODELS=2

# Enable optimizations
export OLLAMA_FLASH_ATTENTION=1
export OLLAMA_NUMA_PREFER=0
```

### Memory Management

```bash
# Monitor memory usage
watch -n 1 'free -h && echo "=== Ollama Process ===" && ps aux | grep ollama'

# Limit model cache
export OLLAMA_MAX_LOADED_MODELS=1

# Use swap if needed (not recommended for production)
sudo swapon --show
```

### Storage Optimization

```bash
# Use SSD for models
sudo mkdir -p /mnt/ssd/ollama/models
sudo chown ollama:ollama /mnt/ssd/ollama/models
export OLLAMA_MODELS=/mnt/ssd/ollama/models

# Clean up unused models
ollama list | grep -v "NAME" | awk '{print $1}' | xargs ollama rm
```

## 9. Backup and Restore

### Model Backup

```bash
#!/bin/bash
# backup-ollama-models.sh

BACKUP_DIR="/var/backups/ollama"
MODELS_DIR="/var/lib/ollama/models"
DATE=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup models directory
tar -czf $BACKUP_DIR/ollama_models_$DATE.tar.gz -C /var/lib/ollama models

# Backup model list
ollama list > $BACKUP_DIR/ollama_models_list_$DATE.txt

echo "Backup completed: $BACKUP_DIR/ollama_models_$DATE.tar.gz"
```

### Configuration Backup

```bash
#!/bin/bash
# backup-ollama-config.sh

BACKUP_DIR="/var/backups/ollama"
DATE=$(date +%Y%m%d_%H%M%S)

# Backup configuration
tar -czf $BACKUP_DIR/ollama_config_$DATE.tar.gz \
    /etc/systemd/system/ollama.service \
    /etc/systemd/system/ollama.service.d/ 2>/dev/null || true

echo "Configuration backup: $BACKUP_DIR/ollama_config_$DATE.tar.gz"
```

### Restore Procedures

```bash
# Restore models
sudo systemctl stop ollama
sudo tar -xzf ollama_models_backup.tar.gz -C /var/lib/ollama
sudo chown -R ollama:ollama /var/lib/ollama/models
sudo systemctl start ollama

# Verify restored models
ollama list
```

### Automated Backup

```bash
# Add to crontab
sudo crontab -e

# Daily model backup at 2 AM
0 2 * * * /opt/ollama/scripts/backup-ollama-models.sh

# Weekly configuration backup
0 3 * * 0 /opt/ollama/scripts/backup-ollama-config.sh
```

## 10. System Requirements

### Minimum Requirements
- **CPU**: 2 cores, 2.0 GHz
- **RAM**: 8GB
- **Storage**: 20GB (small models)
- **Network**: Broadband for model downloads

### Recommended Requirements
- **CPU**: 8+ cores, 3.0+ GHz
- **RAM**: 32GB+
- **GPU**: NVIDIA RTX 3060+ or AMD RX 6600 XT+
- **Storage**: 100GB+ SSD/NVMe
- **Network**: Gigabit for large model downloads

### Model-Specific Requirements

| Model Size | RAM Required | VRAM Required | Storage |
|------------|--------------|---------------|---------|
| 7B         | 8GB          | 4GB           | 4GB     |
| 13B        | 16GB         | 8GB           | 7GB     |
| 30B        | 32GB         | 20GB          | 19GB    |
| 70B        | 64GB         | 48GB          | 39GB    |

## 11. Support

### Official Resources
- **Website**: https://ollama.com
- **GitHub**: https://github.com/ollama/ollama
- **Documentation**: https://github.com/ollama/ollama/tree/main/docs
- **Model Library**: https://ollama.com/library

### Community Support
- **Discord**: https://discord.gg/ollama
- **Reddit**: r/ollama
- **GitHub Issues**: https://github.com/ollama/ollama/issues
- **Discussions**: https://github.com/ollama/ollama/discussions

## 12. Contributing

### How to Contribute
1. Fork the repository on GitHub
2. Create a feature branch
3. Submit pull request
4. Follow Go coding standards
5. Include tests and documentation

### Development Setup
```bash
# Clone repository
git clone https://github.com/ollama/ollama.git
cd ollama

# Install Go dependencies
go mod tidy

# Build from source
go build .

# Run tests
go test ./...
```

## 13. License

Ollama is licensed under the MIT License.

Key points:
- Free to use, modify, and distribute
- Commercial use allowed
- No warranty provided
- Attribution required

## 14. Acknowledgments

### Credits
- **Ollama Team**: Core development team
- **Meta AI**: Llama model family
- **Mistral AI**: Mistral models
- **Community**: Model creators and contributors
- **Hardware Vendors**: NVIDIA, AMD, Apple for acceleration support

## 15. Version History

### Recent Releases
- **v0.3.x**: Latest stable with improved performance
- **v0.2.x**: Added model management improvements
- **v0.1.x**: Initial public release

### Major Features by Version
- **v0.3.0**: Enhanced GPU support, model compression
- **v0.2.0**: REST API improvements, concurrent requests
- **v0.1.0**: Basic model serving and CLI interface

## 16. Appendices

### A. API Usage Examples

#### Basic Chat Completion
```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.1:8b",
  "prompt": "Why is the sky blue?",
  "stream": false
}'
```

#### Streaming Response
```bash
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.1:8b",
  "prompt": "Write a poem about coding",
  "stream": true
}'
```

#### Chat API
```bash
curl http://localhost:11434/api/chat -d '{
  "model": "llama3.1:8b",
  "messages": [
    {
      "role": "user",
      "content": "Hello, how are you?"
    }
  ]
}'
```

### B. Integration Examples

#### Python Integration
```python
import requests
import json

def chat_with_ollama(prompt, model="llama3.1:8b"):
    url = "http://localhost:11434/api/generate"
    data = {
        "model": model,
        "prompt": prompt,
        "stream": False
    }
    
    response = requests.post(url, json=data)
    if response.status_code == 200:
        return response.json()["response"]
    else:
        return "Error: " + str(response.status_code)

# Usage
response = chat_with_ollama("Explain quantum computing")
print(response)
```

#### Node.js Integration
```javascript
const axios = require('axios');

async function chatWithOllama(prompt, model = 'llama3.1:8b') {
    try {
        const response = await axios.post('http://localhost:11434/api/generate', {
            model: model,
            prompt: prompt,
            stream: false
        });
        
        return response.data.response;
    } catch (error) {
        console.error('Error:', error.message);
        return null;
    }
}

// Usage
chatWithOllama('What is machine learning?').then(response => {
    console.log(response);
});
```

### C. Model Customization

#### Creating Custom Models
```bash
# Create Modelfile
cat > Modelfile << 'EOF'
FROM llama3.1:8b

# Set parameters
PARAMETER temperature 0.7
PARAMETER top_p 0.9

# Set system message
SYSTEM """
You are a helpful AI assistant specialized in programming.
Always provide code examples when relevant.
"""
EOF

# Build custom model
ollama create my-coding-assistant -f Modelfile

# Test custom model
ollama run my-coding-assistant "How do I sort a list in Python?"
```

#### Fine-tuning (Advanced)
```bash
# Prepare training data (JSONL format)
cat > training_data.jsonl << 'EOF'
{"prompt": "Question: What is Python?", "completion": "Python is a programming language..."}
{"prompt": "Question: How to install packages?", "completion": "Use pip install package_name..."}
EOF

# Note: Fine-tuning requires additional tools and setup
# Refer to Ollama documentation for detailed fine-tuning guide
```

### D. Performance Monitoring

```bash
#!/bin/bash
# monitor-ollama.sh

echo "=== Ollama Service Status ==="
systemctl status ollama --no-pager

echo -e "\n=== Memory Usage ==="
ps aux | grep ollama | grep -v grep

echo -e "\n=== GPU Usage ==="
if command -v nvidia-smi &> /dev/null; then
    nvidia-smi --query-gpu=utilization.gpu,memory.used,memory.total --format=csv,noheader,nounits
fi

echo -e "\n=== API Health Check ==="
curl -s http://localhost:11434/api/version || echo "API not responding"

echo -e "\n=== Loaded Models ==="
ollama ps

echo -e "\n=== Disk Usage ==="
du -sh /var/lib/ollama/models/*
```

---

For more information and updates, visit https://github.com/howtomgr/ollama