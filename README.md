# Running a Sequencer Node on Aztec Network Testnet  

This guide outlines the steps to set up a sequencer node on the Aztec Network testnet and earn the **Apprentice** role. Follow these instructions to successfully run your node and claim the role. If you encounter issues, check the [Aztec Discord](https://discord.gg/aztec) or official documentation for updates.  

## Prerequisites  

### Hardware Requirements  
- **Recommended**: 8-core CPU, 16 GB RAM, 200 GB+ SSD (for stable, long-term operation)  

### Software Requirements  
- **Ubuntu 22.04** (recommended for Linux users)  
- **Docker** (installed and running)  
- An **Ethereum wallet** with a private key and public address  
- An **Ethereum Sepolia testnet RPC** (e.g., via Alchemy or Infura)  
- A **Beacon RPC** for consensus 

### Network Requirements  
- Stable internet connection  
- Public IP address or proper port forwarding for P2P communication (**ports 40400 TCP/UDP, 8080 TCP**)  
- **Discord account** (to join the Aztec Discord and claim the Apprentice role)  

---

## Step-by-Step Guide  

### 1. Set Up Your Environment  

#### Install Dependencies on Ubuntu  
```bash
sudo apt update -y && sudo apt upgrade -y
sudo apt install curl git
```  

#### Install Docker  
```bash
# Add Docker’s official GPG key:  
sudo apt-get update  
sudo apt-get install ca-certificates curl  
sudo install -m 0755 -d /etc/apt/keyrings  
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc  
sudo chmod a+r /etc/apt/keyrings/docker.asc  

# Add the repository to Apt sources:  
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null  
sudo apt-get update  

# Install Docker  
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin  
```  

#### Install Aztec Tools  
```bash
bash -i <(curl -s https://install.aztec.network)
```  
Restart your terminal to apply changes.  
Verify the installation:  
```bash
aztec
```

Add the Aztec CLI binary path to your shell’s environment (if need)

```bash
echo 'export PATH=$PATH:/root/.aztec/bin' >> ~/.bashrc
source ~/.bashrc
```

Update Aztec Tools docker container to the latest version
```bash
aztec-up alpha-testnet
```

#### Enable Firewall & Open Ports  
```bash
# Firewall  
sudo ufw allow ssh
sudo ufw enable

# Sequencer  
sudo ufw allow 40400
sudo ufw allow 8080
```  

---

### 2. Configure Your Node  

#### Find Your IP  
```bash
curl ifconfig.me
```  
Save the displayed IP address.  

#### RPC URLs  
- Get a **Sepolia testnet RPC URL** (e.g., from Alchemy or Infura).  
- Get a **Beacon RPC URL** (e.g., from [chainstack](https://chainstack.com/) or [Ankr](https://www.ankr.com/rpc/?utm_referral=haxc6QXd36) ).  

#### Set Up a Wallet  
1. Create an Ethereum wallet (e.g., using MetaMask).  
2. Save the **private key** and **public address** securely.  
3. Fund the wallet with **Sepolia testnet ETH** (use a faucet if needed).  

#### Create a `systemd` Service File  
```bash
sudo nano /etc/systemd/system/aztec.service
```  
Paste the following configuration, replacing placeholders (`RPC_URL`, `BEACON_URL`, `0xYourPrivateKey`, `0xYourAddress`, `IP`) with your actual values:  

```ini
[Unit]
Description=Aztec Node Service
After=network.target
Wants=network.target

[Service]
Type=simple
ExecStart=/root/.aztec/bin/aztec start --node --archiver --sequencer \
  --network alpha-testnet \
  --l1-rpc-urls RPC_URL \
  --l1-consensus-host-urls BEACON_URL \
  --sequencer.validatorPrivateKey 0xYourPrivateKey \
  --sequencer.coinbase 0xYourAddress \
  --p2p.p2pIp IP
Restart=always
RestartSec=10
User=root
Group=root
WorkingDirectory=/root/.aztec/alpha-testnet
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target 
```  
Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).


Creat `data` folder for `WorkingDirectory`

```bash
mkdir -p /root/.aztec/alpha-testnet
```  

---

### 3. Run the Sequencer Node  

#### Enable and Start the Service  
```bash
sudo systemctl daemon-reload
sudo systemctl enable aztec.service
sudo systemctl start aztec.service 
```  
The node may take a few minutes to sync.  

#### Verify the Service  
```bash
sudo systemctl status aztec.service
```  
View logs:  
```bash
sudo journalctl -u aztec.service -f
```  

---

### 4. Register as a Validator (Optional)  
To participate as a validator, register with the testnet’s staking contract (later).  

---

### 5. Earn the Apprentice Role  

#### Get the Latest Proven Block Number  
```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getL2Tips","params":[],"id":67}' \
http://localhost:8080 | jq -r ".result.proven.number" 
```  
Save the output (e.g., `20905`).  

#### Generate Your Sync Proof  
```bash
curl -s -X POST -H 'Content-Type: application/json' \
-d '{"jsonrpc":"2.0","method":"node_getArchiveSiblingPath","params":["BLOCK_NUMBER","BLOCK_NUMBER"],"id":67}' \
http://localhost:8080 | jq -r ".result" 
```  
Replace both `BLOCK_NUMBER` entries with the value from above.  

#### Claim the Role on Discord  
1. Join the [Aztec Discord](https://discord.gg/aztec).  
2. Navigate to `operators | start-here`.  
3. Use the `/operator start` command and provide:  
   - `address`: Your validator’s EVM public address.  
   - `block-number`: The block number from earlier.  
   - `proof`: The Base64 string from the sync proof.  

**Verification**: If the node is synced and the proof is valid, you’ll receive the **Apprentice** role instantly.  

![Aztec-Discord](https://github.com/user-attachments/assets/cd8d8e76-6273-48fa-a189-406905160444)

---

### 6. Monitor and Maintain Your Node  

#### Basic Commands  
- **Stop/Restart**:  

```bash
sudo systemctl stop aztec.service  
sudo systemctl restart aztec.service  
```  
- **Delete node data**:  
  ```bash
  rm -rf /root/.aztec/alpha-testnet/data/  
  ```  
- **View logs**:  

```bash
sudo journalctl -u aztec.service -n 100  
```  

#### Updating the Node  

```bash
sudo systemctl stop aztec.service
aztec-up alpha-testnet
sudo systemctl start aztec.service
```  

#### Troubleshooting  
- Ensure **Docker is running** and **ports are open**.  
- Verify **RPC URLs** and **private key** are correct.  
- Check the **Aztec Discord** or **documentation** for support.  

---

## Additional Notes  

- **Costs**: Running locally is free, but a VPS (e.g., Contabo, Hetzner) costs ~$10+/month.  
- **No Official Rewards**: The Apprentice role offers community recognition, not financial incentives.  
- **Testnet Version**: Always use the latest testnet release.  
- **Support**: The Aztec Discord is the best place for real-time help.  

---

## Sources  
- [Aztec Documentation](https://docs.aztec.network/)  
- GitHub Guide by [`0xmoei`](https://github.com/0xmoei/aztec-network)
- Aztec Blog Posts