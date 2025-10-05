# VPN Proxy Docker

Docker Compose setup for running a VPN gateway with Privoxy filtering. Route your network traffic through a VPN while filtering tracking parameters and unwanted content.

## Features

- **VPN Gateway**: Uses [Gluetun](https://github.com/qdm12/gluetun) to route all traffic through a VPN
- **Privacy Filtering**: Privoxy filters tracking parameters (utm_*, fbclid, gclid, etc.)
- **Network Proxy**: Expose a proxy server on your local network for all devices
- **Container Networking**: Privoxy shares the VPN container's network, ensuring all traffic goes through VPN

## Prerequisites

- Docker and Docker Compose installed
- A WireGuard VPN configuration file (from your VPN provider)
  - Supported providers: Mullvad, NordVPN, ProtonVPN, and [many more](https://github.com/qdm12/gluetun-wiki/tree/main/setup/providers)
  - Or use any custom WireGuard config

## Setup

### 1. Clone this repository

```bash
git clone https://github.com/yourusername/vpn-proxy-docker.git
cd vpn-proxy-docker
```

### 2. Add your WireGuard configuration

Create a `wireguard` directory and add your VPN config:

```bash
mkdir -p wireguard
# Copy your WireGuard config file
cp /path/to/your/config.conf wireguard/wg0.conf
```

See `wireguard/wg0.conf.example` for the expected format.

### 3. Configure Privoxy (Optional)

The default configuration in `privoxy-config/` includes:
- Tracking parameter removal (utm_*, fbclid, gclid)
- Basic privacy settings

Customize `privoxy-config/config` to adjust:
- Access control (permit-access/deny-access)
- Debug level
- Buffer limits

Add custom filters in `privoxy-config/user.filter` and actions in `privoxy-config/user.action`.

### 4. Update docker-compose.yml

Edit `docker-compose.yml` to set your desired proxy port:

```yaml
ports:
  # Change this to your server's IP address if needed
  - "8118:8118"  # or "YOUR_SERVER_IP:8118:8118"
```

### 5. Start the services

```bash
docker-compose up -d
```

## Usage

### Configure Devices to Use the Proxy

Point your devices to use the proxy server:
- **Proxy Address**: Your Docker host IP (e.g., `192.168.1.100`)
- **Proxy Port**: `8118`

#### Examples:

**macOS/iOS:**
Settings → Wi-Fi → Configure Proxy → Manual
- Server: `192.168.1.100`
- Port: `8118`

**Windows:**
Settings → Network & Internet → Proxy → Manual proxy setup

**Linux/Firefox:**
Preferences → Network Settings → Manual proxy configuration
- HTTP Proxy: `192.168.1.100`
- Port: `8118`
- Check "Use this proxy server for all protocols"

### Verify VPN Connection

Check your public IP to confirm traffic is routed through the VPN:

```bash
curl --proxy http://localhost:8118 https://ifconfig.me
```

### Monitor Logs

Watch Privoxy filtering in action:
```bash
docker-compose logs -f privoxy
```

Check VPN connection status:
```bash
docker-compose logs -f vpn
```

## Useful Commands

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# Restart after config changes
docker-compose restart privoxy

# View logs
docker-compose logs -f

# Check status
docker-compose ps
```

## Advanced Configuration

### Using a Supported VPN Provider

Instead of custom WireGuard config, you can use Gluetun's built-in provider support:

```yaml
environment:
  - VPN_SERVICE_PROVIDER=mullvad  # or nordvpn, protonvpn, etc.
  - VPN_TYPE=wireguard
  - WIREGUARD_PRIVATE_KEY=your_private_key
  - WIREGUARD_ADDRESSES=your_address
  - SERVER_CITIES=Seattle  # or other city
```

See [Gluetun documentation](https://github.com/qdm12/gluetun-wiki) for provider-specific setup.

### Custom Privoxy Filters

Add custom URL filters in `privoxy-config/user.filter`:

```
FILTER: ads Remove advertising domains
s|ads\.example\.com||g
```

Then enable in `privoxy-config/user.action`:

```
{+filter{ads}}
.*
```

### Network Access Control

Restrict which networks can use the proxy in `privoxy-config/config`:

```
# Allow only your local network
permit-access 192.168.1.0/24

# Deny specific networks
deny-access 192.168.100.0/24
```

## Troubleshooting

### VPN not connecting
- Check your WireGuard config is valid: `docker-compose logs vpn`
- Ensure the config file is mounted correctly
- Verify your VPN provider credentials/keys are correct

### Proxy not accessible
- Check Docker host firewall allows port 8118
- Verify port binding in docker-compose.yml
- Ensure containers are running: `docker-compose ps`

### Privoxy not filtering
- Check Privoxy logs: `docker-compose logs privoxy`
- Verify filter files are mounted: `docker-compose exec privoxy ls /etc/privoxy`
- Restart after config changes: `docker-compose restart privoxy`

## Security Considerations

- **Never commit your WireGuard config** - it contains private keys
- Keep `.gitignore` updated to exclude sensitive files
- Use strong access controls in Privoxy config
- Regularly update Docker images: `docker-compose pull`

## License

MIT

## Contributing

Pull requests welcome! Please ensure your changes maintain privacy and security best practices.
