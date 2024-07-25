
# One-Click Ethereum Classic (ETC) Mining Script

This repository provides a one-click script to set up Ethereum Classic (ETC) mining using T-Rex Miner. The script includes GPU detection, logging, automatic retries, and dependency installation, making it easy to start mining with minimal setup.

## Features
- **Automated Setup**: Installs necessary dependencies and configures the mining environment.
- **GPU Detection**: Automatically detects and uses available NVIDIA GPUs.
- **Logging**: Logs mining activity and errors for monitoring and troubleshooting.
- **Retry Mechanism**: Automatically restarts the mining process in case of crashes.

## Prerequisites
- **Ubuntu 20.04 or later**
- **NVIDIA GPUs with CUDA support**
- **NVIDIA Drivers and CUDA Toolkit**

## How to Get Started

### 1. Get an Ethereum Classic (ETC) Wallet Address
You need an Ethereum Classic (ETC) wallet address to receive your mining rewards. You can use Trust Wallet to create one:

- [Trust Wallet - How to Get an Ethereum Classic Address](https://trustwallet.com/ethereum-classic-wallet)

### 2. Choose a Mining Pool
This script is configured to use the 2Miners pool by default. For more information about 2Miners:

- [2Miners ETC Pool](https://2miners.com/etc-mining-pool)


### 4. Configure the Script
Create a file called `mine.sh` script to include your ETC wallet address and worker name:

```sh
POOL="stratum+tcp://us-etc.2miners.com:1010"
WALLET="your_etc_wallet_address" # Replace with your wallet address
WORKER_NAME="your_worker_name" # Choose your worker name
LOG_FILE="/var/log/etc_miner.log" # Log file location
RETRY_INTERVAL=10  # seconds
MAX_RETRIES=5
TREX_URL="https://github.com/trexminer/T-Rex/releases/download/0.25.15/t-rex-0.25.15-linux.tar.gz" # Adjust to latest version
```

## Script Details

### mine.sh

```sh
#!/bin/bash

# Constants
POOL="stratum+tcp://us-etc.2miners.com:1010"
WALLET="your_wallet_address" # Replace with your wallet address
WORKER_NAME="rig1" # Choose your worker name
LOG_FILE="/var/log/etc_miner.log" # Log file location
RETRY_INTERVAL=10  # seconds
MAX_RETRIES=5
TREX_URL="https://github.com/trexminer/T-Rex/releases/download/0.25.15/t-rex-0.25.15-linux.tar.gz" # Adjust to latest version

# Logging function
log() {
    local message="$1"
    echo "$(date +"%Y-%m-%d %H:%M:%S") : $message" | tee -a "$LOG_FILE"
}

# Function to detect GPUs
detect_gpus() {
    log "Detecting GPUs..."
    GPUS=$(nvidia-smi --query-gpu=index --format=csv,noheader | tr '\n' ',' | sed 's/,$//')
    if [ -z "$GPUS" ]; then
        log "No GPUs detected. Exiting."
        exit 1
    fi
    log "Detected GPUs: $GPUS"
}

# Function to install dependencies and T-Rex Miner
install_dependencies() {
    log "Installing dependencies..."
    sudo apt-get update
    sudo apt-get install -y build-essential cmake git cuda nvidia-cuda-toolkit

    log "Downloading T-Rex Miner..."
    wget -O t-rex.tar.gz $TREX_URL
    tar -xvf t-rex.tar.gz
    sudo mv t-rex /usr/local/bin/
    rm t-rex.tar.gz
}

# Function to start mining
start_mining() {
    log "Starting mining process..."
    /usr/local/bin/t-rex -a etchash -o $POOL -u $WALLET -p x -w $WORKER_NAME --devices $GPUS 2>&1 | tee -a "$LOG_FILE"
}

# Function to monitor and restart mining process if necessary
monitor_mining() {
    local retries=0
    while [ $retries -lt $MAX_RETRIES ]; do
        start_mining
        if [ $? -eq 0 ]; then
            log "Mining process exited successfully."
            break
        else
            log "Mining process crashed. Attempting restart ($((retries + 1))/$MAX_RETRIES)..."
            retries=$((retries + 1))
            sleep $RETRY_INTERVAL
        fi
    done

    if [ $retries -eq $MAX_RETRIES ]; then
        log "Maximum retry attempts reached. Exiting."
        exit 1
    fi
}

# Main script execution
log "Ethereum Classic mining script started."
detect_gpus
install_dependencies
monitor_mining
log "Ethereum Classic mining script stopped."
```

### 4. Run the Script
Make the script executable and run it:

```bash
chmod +x mine.sh
sudo ./mine.sh
```

## Monitoring and Troubleshooting

### Checking Logs
You can view the log file to monitor the mining process and troubleshoot any issues:

```bash
tail -f /var/log/etc_miner.log
```

### Monitoring Your Mining Stats
Visit the 2Miners dashboard and enter your wallet address to monitor your mining statistics:

- [2Miners ETC Pool Dashboard](https://2miners.com/etc-mining-pool)

## Additional Resources

- [Ethereum Classic (ETC) Official Website](https://ethereumclassic.org/)
- [Trust Wallet - How to Get an Ethereum Classic Address](https://trustwallet.com/ethereum-classic-wallet)
- [2Miners ETC Pool](https://2miners.com/etc-mining-pool)
- [T-Rex Miner GitHub Repository](https://github.com/trexminer/T-Rex)

## License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Disclaimer
Mining cryptocurrency involves financial risk and may not always be profitable. Ensure you understand the risks involved and monitor your mining operation to avoid losses.

### Summary

This README file provides a comprehensive guide to setting up and running the one-click ETC mining script, including steps to obtain a wallet address, configure the script, and monitor the mining process. It also includes links to relevant resources for getting an ETC wallet and monitoring mining stats.
