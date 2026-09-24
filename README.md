# Introduction
Base Level Project to create a workplace for agentic flow.

Create and EC2 Instance (Ubuntu 24.04 LTS with T3.Large)

And allow remote vscode deployment / access. 

To approaches:

* Official [vscode-server](https://code.visualstudio.com/docs/remote/vscode-server)
* Open Source Browser Mode [code-server](https://github.com/coder/code-server)


## Ec2 User Data Field


#!/bin/bash
set -euxo pipefail

# 1. Configuration - Replace these values before launch
GH_ACCESS_TOKEN="ghp_yourPersonalAccessTokenHere"
# Name less than 20 Chars
TUNNEL_NAME="ec2-dev-greenwbe"
TARGET_USER="ubuntu" # Use 'ec2-user' for Amazon Linux / RHEL

# 2. System updates and required dependencies
export DEBIAN_FRONTEND=noninteractive
if command -v apt-get &> /dev/null; then
    apt-get update -y
    apt-get install -y curl tar tmux jq
elif command -v dnf &> /dev/null; then
    dnf install -y curl tar tmux jq
elif command -v yum &> /dev/null; then
    yum install -y curl tar tmux jq
fi

# 3. Download and unpack official VS Code standalone CLI
ARCH=$(uname -m)
if [ "$ARCH" = "x86_64" ]; then
    CLI_URL="https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-x64"
elif [ "$ARCH" = "aarch64" ]; then
    CLI_URL="https://code.visualstudio.com/sha/download?build=stable&os=cli-alpine-arm64"
fi

curl -sSL "$CLI_URL" -o /tmp/vscode_cli.tar.gz
tar -xzf /tmp/vscode_cli.tar.gz -C /usr/local/bin/
chmod +x /usr/local/bin/code
rm -f /tmp/vscode_cli.tar.gz

# 4. Create persistent systemd service for headless tunnel execution
TARGET_HOME=$(eval echo "~$TARGET_USER")

cat <<EOF > /etc/systemd/system/vscode-tunnel.service
[Unit]
Description=Visual Studio Code Remote Tunnel
After=network.target network-online.target
Wants=network-online.target

[Service]
Type=simple
User=${TARGET_USER}
WorkingDirectory=${TARGET_HOME}
Environment="VSCODE_CLI_USE_DESKTOP_KEYRING=0"
Environment="VSCODE_CLI_REQUIRE_TOKEN=${GH_ACCESS_TOKEN}"
ExecStart=/usr/local/bin/code tunnel --accept-server-license-terms --name ${TUNNEL_NAME}
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

# 5. Reload systemd, enable and launch service
systemctl daemon-reload
systemctl enable vscode-tunnel.service
systemctl start vscode-tunnel.service

# ---------------------------------------------------
# launch
https://vscode.dev/tunnel/ec2-ben-workspace

Check the service:

Sep 24 14:21:25 ip-172-31-16-244 code[2716]: *
Sep 24 14:21:25 ip-172-31-16-244 code[2716]: [2026-09-24 14:21:25] info Using GitHub for authentication, run `code tunnel user login --provider <provider>` option to change this.
Sep 24 14:21:25 ip-172-31-16-244 code[2716]: To grant access to the server, please log into https://github.com/login/device and use code 730F-9AFB

Also Check
Sep 24 14:24:40 ip-172-31-16-244 code[2779]: [2026-09-24 14:24:40] info Names cannot be longer than 20 characters. Please try a different name. is an invalid name
Sep 24 14:24:40 ip-172-31-16-244 code[2779]: [2026-09-24 14:24:40] error invalid name: Names cannot be longer than 20 characters. Please try a different name.