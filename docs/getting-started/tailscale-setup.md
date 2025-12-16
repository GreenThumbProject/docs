# Tailscale Setup

This guide explains how to set up Tailscale on your Raspberry Pi for remote access during development.

## What is Tailscale?

Tailscale is a VPN service that creates a secure network (tailnet) between your devices. It provides:

- **Always-on IP address** - Access your Pi from anywhere without port forwarding
- **Secure SSH access** - Connect to your Pi without exposing it to the internet
- **Easy development** - Code and deploy from anywhere

!!! tip "Future Vision"
    Currently used for development. In the future, we plan to enable clients to access their greenhouses remotely without needing to understand VPNs.

## Installation

### 1. Install Tailscale on Raspberry Pi

```bash
# Add Tailscale's package signing key and repository
curl -fsSL https://tailscale.com/install.sh | sh
```

### 2. Start Tailscale

```bash
sudo tailscale up
```

This will display a URL. Open it in your browser to authenticate with your Tailscale account.

### 3. Verify Connection

```bash
tailscale status
```

You should see your device listed with a `100.x.x.x` IP address.

## SSH Configuration

### Enable SSH on Raspberry Pi

```bash
# Enable SSH service
sudo systemctl enable ssh
sudo systemctl start ssh

# Verify SSH is running
sudo systemctl status ssh
```

### Configure SSH Keys (Recommended)

From your **local machine**:

```bash
# Generate SSH key if you don't have one
ssh-keygen -t ed25519 -C "your-email@example.com"

# Copy your public key to the Pi
ssh-copy-id pi@<tailscale-ip>
```

### Connect via SSH

```bash
# Using Tailscale IP
ssh pi@100.x.x.x

# Or using MagicDNS (if enabled)
ssh pi@raspberrypi
```

## Enable Tailscale SSH (Optional)

Tailscale can manage SSH authentication directly:

```bash
# On the Raspberry Pi
sudo tailscale up --ssh
```

This enables SSH access using your Tailscale identity without managing SSH keys.

## Development Workflow

With Tailscale configured, you can:

1. **Remote development** - Use VS Code Remote SSH to edit code directly on the Pi
2. **Deploy from anywhere** - Push to GitHub and let CI/CD deploy to your Pi
3. **Access the dashboard** - Open `http://<tailscale-ip>:8080` from any device in your tailnet

## Tailscale Admin Console

Manage your devices at [login.tailscale.com/admin](https://login.tailscale.com/admin):

- View all connected devices
- Enable/disable MagicDNS for easy hostnames
- Configure access controls (ACLs)
- Monitor connection status

## Troubleshooting

### Tailscale Not Starting

```bash
# Check service status
sudo systemctl status tailscaled

# Restart the service
sudo systemctl restart tailscaled
```

### SSH Connection Refused

```bash
# Verify SSH is running
sudo systemctl status ssh

# Check firewall (if enabled)
sudo ufw status
sudo ufw allow ssh
```

### Device Not Appearing in Tailnet

1. Re-authenticate: `sudo tailscale up`
2. Check internet connectivity
3. Verify account at [tailscale.com](https://tailscale.com)

## Next Steps

- [Installation](installation.md) - Set up GreenThumb services
- [Configuration](configuration.md) - Configure your environment
