# Introduction
Base Level Project to create a workplace for agentic flow.

# On Restart - OpenCode launch
```
export $(grep -v '^#' .env | xargs) && opencode
```

# Running Whilst Detaching
```
tmux new -s opencode-work
export $(grep -v '^#' .env | xargs)
opencode
```
Close using ctrl+b, then d (to close cleanly)


On resume
```
tmux attach -t opencode-work
```

# Setup

And allow remote vscode deployment / access. 

Two approaches possible:

* Official [vscode-server](https://code.visualstudio.com/docs/remote/vscode-server)
* Open Source Browser Mode [code-server](https://github.com/coder/code-server)

This will use the official MS vscode-server


Create and EC2 Instance (Ubuntu 24.04 LTS with T3.Large), set disk space to 60GB then in (advanced) User Data field use (remembering to change the GH_ACCESS_TOKEN and the TUNNEL_NAME

## Ec2 User Data Field

```
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

# Additional Settings
# Node/NPM
sudo DEBIAN_FRONTEND=noninteractive apt-get update -y
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y ca-certificates curl gnupg

# Add NodeSource repository for the LTS release
sudo curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -

# Install Node.js and npm without prompts
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold" nodejs

echo "Installing OpenCode and agent-browser..."
sudo npm install -g opencode-ai 
sudo npm install -g agent-browser

# https://github.com/trailhq/Graft
# echo "Adding graft"
# npm install -g @nanonets/graft   # install the CLI, once

echo "Installing Chromium..."
# Chrome for Testing has no Linux ARM64 builds, so use Debian's chromium on all
# architectures; agent-browser finds it via AGENT_BROWSER_EXECUTABLE_PATH (containerEnv).
sudo find /etc/apt/sources.list.d -maxdepth 1 -type f -iname '*yarn*' -delete
sudo apt-get update
sudo apt-get install -y chromium

echo "Adding the agent-browser skill for OpenCode..."
sudo npx -y skills add vercel-labs/agent-browser -a opencode -y

sudo apt-get install -y openjdk-25-jdk

## Install Maven and SQLite CLI
sudo apt-get install -y maven

sudo apt-get install -y sqlite3

echo "Installing envoy"
sudo curl -sL -o /usr/bin/envoy https://github.com/envoyproxy/envoy/releases/download/v1.32.1/envoy-1.32.1-linux-x86_64
sudo chmod +x /usr/bin/envoy


```
# ---------------------------------------------------
# launch
Goto a webbrowser and use the $TUNNEL_NAME for the workspace
https://vscode.dev/tunnel/ec2-ben-workspace

Check the service, if issues: 
sudo journalctl -u vscode-tunnel.service -n 50 --no-pager

Some issues seen:
```
Sep 24 14:21:25 ip-172-31-16-244 code[2716]: *
Sep 24 14:21:25 ip-172-31-16-244 code[2716]: [2026-09-24 14:21:25] info Using GitHub for authentication, run `code tunnel user login --provider <provider>` option to change this.
Sep 24 14:21:25 ip-172-31-16-244 code[2716]: To grant access to the server, please log into https://github.com/login/device and use code 730F-9AFB

Also Check
Sep 24 14:24:40 ip-172-31-16-244 code[2779]: [2026-09-24 14:24:40] info Names cannot be longer than 20 characters. Please try a different name. is an invalid name
Sep 24 14:24:40 ip-172-31-16-244 code[2779]: [2026-09-24 14:24:40] error invalid name: Names cannot be longer than 20 characters. Please try a different name.
```



# Agentic Development
.env should include:  (TODO .env was when use devcontainers, but now moving to own ec2 instance)
```
AWS_BEARER_TOKEN_BEDROCK=
AWS_REGION=us-east-1
```
