# Pipe-network
Download the Node Client Binary
Create Directory:

Copy
sudo mkdir -p /opt/dcdn
Download Pipe tool Binary from URL:

Copy
sudo curl -L "$PIPE-URL" -o /opt/dcdn/pipe-tool
Download Node Binary from URL:

Copy
sudo curl -L "$DCDND-URL" -o /opt/dcdn/dcdnd
Make Binary Executable:

Copy
sudo chmod +x /opt/dcdn/pipe-tool
sudo chmod +x /opt/dcdn/dcdnd
Setup the dcdnd node systemd service
To configure the  systemd service, follow these steps:

Create the Service File:
Save your service definition in a .service file within the /etc/systemd/system/ directory.

Sample Service File:

Copy
# Create service file using cat
sudo cat > /etc/systemd/system/dcdnd.service << 'EOF'
[Unit]
Description=DCDN Node Service
After=network.target
Wants=network-online.target

[Service]
# Path to the executable and its arguments
ExecStart=/opt/dcdn/dcdnd \
                --grpc-server-url=0.0.0.0:8002 \
                --http-server-url=0.0.0.0:8003 \
                --node-registry-url="https://rpc.pipedev.network" \
                --cache-max-capacity-mb=1024 \
                --credentials-dir=/root/.permissionless \
                --allow-origin=*

# Restart policy
Restart=always
RestartSec=5

# Resource and file descriptor limits
LimitNOFILE=65536
LimitNPROC=4096

# Logging
StandardOutput=journal
StandardError=journal
SyslogIdentifier=dcdn-node


# Working directory
WorkingDirectory=/opt/dcdn

[Install]
WantedBy=multi-user.target
EOF
Optional: Enhanced Security
For enhanced security, run dcdnd.service under a dedicated service account. This is a more advanced configuration.

Copy
# Create dedicated service account:
sudo useradd -r -m -s /sbin/nologin dcdn-svc-user -d /home/dcdn-svc-user
# Create token subfolder
sudo mkdir -p /home/dcdn-svc-user/.permissionless
sudo chown -R dcdn-svc-user:dcdn-svc-user /home/dcdn-svc-user/.permissionless
# Update the .service file [Service] to include:
User=dcdn-svc-user
Group=dcdn-svc-user
# Update the credentials path under [Service] accordingly:    
--credentials-dir=/home/dcdn-svc-user/.permissionless \
# Move the previously generated registration token
mv /root/.permissionless/registration_token.json /home/dcdn-svc-user/.permissionless
Ports to open
The ports the user needs to open based on the provided service file are:

Port 8002: This is for the gRPC server (--grpc-server-url=0.0.0.0:8002).

Port 8003: This is for the HTTP server (--http-server-url=0.0.0.0:8003).

On UFW (Ubuntu/Debian):

Copy
sudo ufw allow 8002/tcp
sudo ufw allow 8003/tcp
sudo ufw reload
Additional Notes:
Ensure these ports are opened in any cloud provider's network security settings (e.g., AWS Security Groups, Azure Network Security Groups, etc.), if the server is hosted in the cloud.

Make sure no other service is already using these ports to avoid conflicts.

To load, enable and start the new service, follow these steps:
Reload Systemd Daemon: Use this command to reload the systemd manager configuration.

Copy
sudo systemctl daemon-reload
Enable Service at Boot: Configure the service to start automatically upon system boot.

Copy
sudo systemctl enable dcdnd
Start the Service: Manually start the service.

Copy
sudo systemctl start dcdnd
Configuration and management of the new service are now controlled

Recap and explanation:
The steps outlined above guide you through configuring a systemd service for a DCDN Node Service. By creating a service file in /etc/systemd/system/, you define how the service starts and stops, as well as setting necessary parameters like the executable path and network dependencies. After creating the file, you reload the systemd daemon to apply changes, start the service to check its functionality, and enable it to ensure it automatically starts at boot. Lastly, the service's status is verified to confirm it is running as expected.

Follow these steps to manage the state and configuration of the dcdnd service

Managing the dcdnd Service
1. View Logs

To view the logs for the dcdnd service in real-time, use:

Copy
sudo journalctl -f -u dcdnd.service
2. Restart Service

To restart the dcdnd service after making configuration changes, execute:

Copy
sudo systemctl restart dcdnd
3. Stop Service

If you need to stop the dcdnd service, use:

Copy
sudo systemctl stop dcdnd
4. Check Status

To check the current status of the dcdnd service, run:

Copy
systemctl status dcdnd
Pipe Network Wallet Setup & Registration
Log In to Pipe Network
To log in to the Pipe Network, execute the following command:

Copy
pipe-tool login --node-registry-url="https://rpc.pipedev.network"
Generate and Register Wallet
You can either generate and register a new wallet or link an existing wallet

Generate a new wallet (Solana Keypair):

To generate a Solana keypair, run the following command:

Copy
pipe-tool generate-wallet --node-registry-url="https://rpc.pipedev.network"
You will be prompted to enter an optional 12-word passphrase

Keypair Location:
By default, the keypair is saved to ~/.permissionless/key.json. To save the keypair to a different location, specify the desired file path:
--key-path=<path to save the keypair file>

To ensure the security of your wallet, it's crucial to back up your 12-word recovery phrase and key file in a secure location. Failure to do so may result in losing access if the key file or recovery phrase is compromised. When you run the command successfully, you'll receive the 12-word recovery phrase, the public key of the keypair, and the location where the file is stored. The public key is sent to our pipe.network backend as specified by the –node-registry-url

Linking Your Existing Wallet
Alternatively you can link your existing wallet’s public address (generated by solana-keygen or other wallets that support Solana) to your account.  Two different methods are described:

Using Base58 Encoded Public Key
To link your wallet using a base58 encoded public key, run the following command:

Copy
pipe-tool link-wallet --node-registry-url="https://rpc.pipedev.network" --public-key="<base58 encoded public key>"
This command will look for the key.json file at the default location ~/.permissionless/key.json. You can specify a custom path to your key file using:

Copy
--key-path="<path to file>"
Linking Wallet Using Keypair File
To link your wallet's public address with a keypair file, execute:

Copy
pipe-tool link-wallet --node-registry-url="https://rpc.pipedev.network"
Conclusion
Your node should be functioning at this point, here are a few ways to test it out:
1. pipe-tool list-nodes --node-registry-url="https://rpc.pipedev.network"
2. Simply confirm the ports are open and traffic can get to your node over the internet:
a) Use a web based port tester to test against your public IP on port 8002 & 8003
OR
b) Have a friend at another location try 'telnet <YOUR-PUBLIC-IP> 8002' and 'telnet <YOUR-PUBLIC-IP> 8003' to see if an initial connection is made

Appendix
Wallet Utilities
View Private Key
To view your wallet’s private address, run the following command:

Copy
pipe-tool show-private-key
You can also specify a custom key file location using:

Copy
--key-path="<path to key file>"
By default, the key file is located at ~/.permissionless/key.json.

View Public Key
To view your wallet’s public address, use the command:

Copy
pipe-tool show-public-key
Similarly, you can specify a custom key file location:

Copy
--key-path="<path to key file>"
The default location for the key file is ~/.permissionless/key.json.

View Linked Wallet
To view the wallet linked to your account, execute the following command:

Copy
pipe-tool link-wallet --show-linked --node-registry-url="https://rpc.pipedev.network"
pipe-tool Command Appendix
Command pipe-tool login
