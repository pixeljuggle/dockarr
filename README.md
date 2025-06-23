# Media Server Docker Compose Setup

This repository contains a Docker Compose setup for a media server, including services like VPN, qBittorrent, Jackett, Radarr, Sonarr, Jellyseerr, and Jellyfin. Follow the instructions below to set up and run the media server on your system.

## Prerequisites

- A Linux-based system (e.g., Ubuntu, Debian) or any system that supports Docker.
- Basic knowledge of the terminal and Docker.

## Step 1: Install Docker

To install Docker, follow the official Docker installation guide for your operating system:

- [Docker Installation Documentation](https://docs.docker.com/get-docker/)

For example, on Ubuntu:

#### Uninstall old versions

Before you can install Docker Engine, you need to uninstall any conflicting packages.

Your Linux distribution may provide unofficial Docker packages, which may conflict with the official packages provided by Docker. You must uninstall these packages before you install the official version of Docker Engine.

```bash
for pkg in docker.io docker-doc docker compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

#### Install using the convenience script

This example downloads the script from https://get.docker.com/ and runs it to install the latest stable release of Docker on Linux:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

#### Verify the installation:

```bash
docker --version
```

## Step 2: Install Docker Compose

To install Docker Compose, follow the official Docker Compose installation guide:

- [Docker Compose Installation Documentation](https://docs.docker.com/compose/install/)

For example, on Ubuntu:

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

Add the `docker` group

```bash
sudo groupadd docker
```

Add your user to the `docker` group.

```bash
sudo usermod -aG docker $USER
```

If you're running Linux in a virtual machine, it may be necessary to restart the virtual machine for changes to take effect.

Verify the installation:

```bash
docker compose --version
```

```bash
docker run hello-world
```

Configure Docker to start on boot with systemd

```bash
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```

## Step 3: Clone the Repository

Clone this repository to your local machine:

```bash
git clone <repository-url>
cd dockarr
```

## Step 4: Configure Environment Variables

Edit the `.env` file to set your user ID, group ID, timezone, and folder paths. Use the following command to find your user ID and group ID:

```bash
id $(whoami)
```

Update the `.env` file accordingly:

```properties
PUID=1001
PGID=1002
TZ=Europe/Paris
APPDATA_FOLDER="./appdata"
DOWNLOAD_FOLDER="./downloads"
MOVIE_FOLDER="./media/movies"
TV_FOLDER="./media/tv"
```

**Folder Structure Explanation:**

- `appdata/`: Contains configuration files for all services
- `downloads/`: Temporary storage for downloaded content
- `downloads/movies/`: Downloaded movies before processing
- `downloads/tv/`: Downloaded TV shows before processing
- `downloads/incomplete/`: Partial downloads
- `downloads/torrents/`: Torrent files (optional)
- `media/movies/`: Final organized movie library
- `media/tv/`: Final organized TV show library

## Step 5: Configure VPN

**Important**: This setup uses Gluetun as the VPN client. All torrent traffic will be routed through your VPN connection for privacy and security.

### VPN Configuration Options:

**Option 1: Custom VPN (Recommended for flexibility)**

1. Place your OpenVPN configuration file in the `vpn` folder.
2. Update the `.env` file:
   ```properties
   VPN_SERVICE_PROVIDER=custom
   VPN_TYPE=openvpn
   OPENVPN_CUSTOM_CONFIG="/gluetun/vpn/your-vpn-config.ovpn"
   ```

**Option 2: Built-in VPN Providers**
For popular VPN providers like NordVPN, ExpressVPN, Surfshark, etc., you can use built-in configurations:

1. Update the `.env` file (example for NordVPN):

   ```properties
   VPN_SERVICE_PROVIDER=nordvpn
   VPN_TYPE=openvpn
   NORDVPN_USER=your_username
   NORDVPN_PASSWORD=your_password
   SERVER_REGIONS=United States
   ```

2. For other providers, check the [Gluetun documentation](https://github.com/qdm12/gluetun/wiki) for specific configuration options.

**VPN Kill Switch**: The setup includes a kill switch - if the VPN connection drops, all torrent traffic will stop to prevent IP leaks.

**Testing VPN Connection**:
After starting the containers, verify your VPN is working:

```bash
# Check qBittorrent's IP address (should be your VPN IP)
docker exec qbittorrent curl -s ifconfig.me
```

## Step 6: Start the Services

Run the following command to start all services:

```bash
docker compose up -d
```

This will start all the containers in detached mode.

## Step 7: Access the Services

- **qBittorrent**: `http://<your-ip>:8900`
- **Jackett**: `http://<your-ip>:9117`
- **Radarr**: `http://<your-ip>:7878`
- **Sonarr**: `http://<your-ip>:8989`
- **Jellyseerr**: `http://<your-ip>:5056`
- **Jellyfin**: `http://<your-ip>:8096`

Replace `<your-ip>` with your server's IP address.

---

## Service Setup & Configuration

After starting the containers, follow these steps to configure each service. All services can communicate using their container names as hostnames (e.g., `qbittorrent`, `jackett`).

### qBittorrent ([Docs](https://docs.linuxserver.io/images/docker-qbittorrent/))

**Initial Setup:**

1. Access the Web UI at `http://<your-ip>:8900`.
2. Get the temporary password:
   ```bash
   docker logs qbittorrent | grep "WebUI admin password"
   ```
   The password will be shown in the logs on first startup.
3. Login with username `admin` and the temporary password.
4. **IMPORTANT**: Immediately change the credentials in Web UI → Tools → Options → Web UI:
   - Change username from `admin` to something secure
   - Set a strong password
   - Enable "Bypass authentication for clients on localhost" (optional)
5. If you don't change the password, it will reset on every container restart.

**Download Configuration:**

1. Go to Tools → Options → Downloads:
   - Default Save Path: `/downloads/` (already configured)
   - Keep incomplete torrents in: `/downloads/incomplete/` (already configured)
   - Copy .torrent files to: `/downloads/torrents/` (optional)
2. Go to Tools → Options → BitTorrent:
   - Enable DHT, PeX, and Local Peer Discovery
   - Set appropriate upload/download limits

**Categories (Required for Radarr/Sonarr):**

1. Right-click in the category panel → Add category
2. Create categories: `movies`, `tv`, `music` (as needed)
3. Set save paths:
   - movies: `/downloads/movies/`
   - tv: `/downloads/tv/`

### Jackett ([Docs](https://github.com/Jackett/Jackett/wiki))

**Initial Setup:**

1. Access at `http://<your-ip>:9117`.
2. Copy the API Key from the top-right corner for later use.

**Adding Indexers:**

1. Click "Add indexer" and search for your trackers.
2. For public trackers (like 1337x, RARBG alternatives):
   - Click the wrench icon → Configure
   - Test the configuration
3. For private trackers:
   - Enter your username and password or API key
   - Test the configuration
4. Popular public indexers to add:
   - YTS (movies)
   - EZTV (TV shows)
   - LimeTorrents
   - TorrentGalaxy

**Get Torznab URLs:**

- After adding indexers, copy the Torznab feed URL for each
- Format: `http://jackett:9117/api/v2.0/indexers/<indexer-id>/results/torznab/`

### Radarr ([Docs](https://wiki.servarr.com/en/radarr))

**Initial Setup:**

1. Access at `http://<your-ip>:7878`.
2. Complete the initial setup wizard.

**Configure Download Client:**

1. Go to Settings → Download Clients → Add (+) → qBittorrent:
   - Name: `qBittorrent`
   - Enable: ✓
   - Host: `qbittorrent`
   - Port: `8900`
   - Username: (as set in qBittorrent)
   - Password: (as set in qBittorrent)
   - Category: `movies`
   - Test and Save

**Configure Indexers:**

1. Go to Settings → Indexers → Add (+) → Torznab → Custom:
   - Name: (indexer name from Jackett)
   - Enable RSS Sync: ✓
   - Enable Interactive Search: ✓
   - URL: `http://jackett:9117/api/v2.0/indexers/<indexer-id>/results/torznab/`
   - API Key: (from Jackett)
   - Categories: 2000,2010,2020,2030,2040,2045,2050,2060 (movies)
   - Test and Save
2. Repeat for each indexer from Jackett.

**Configure Media Management:**

1. Go to Settings → Media Management:
   - Enable "Rename Movies": ✓
   - Root Folders: Add `/movies`
   - Movie Naming: Configure as desired
   - Permissions: Set CHMOD folder to `755`

**Quality Profiles:**

1. Go to Settings → Profiles:
   - Create/modify quality profiles as needed
   - Common setup: HD-1080p (allow 720p/1080p, prefer 1080p)

### Sonarr ([Docs](https://wiki.servarr.com/en/sonarr))

**Initial Setup:**

1. Access at `http://<your-ip>:8989`.
2. Complete the initial setup wizard.

**Configure Download Client:**

1. Go to Settings → Download Clients → Add (+) → qBittorrent:
   - Name: `qBittorrent`
   - Enable: ✓
   - Host: `qbittorrent`
   - Port: `8900`
   - Username: (as set in qBittorrent)
   - Password: (as set in qBittorrent)
   - Category: `tv`
   - Test and Save

**Configure Indexers:**

1. Go to Settings → Indexers → Add (+) → Torznab → Custom:
   - Name: (indexer name from Jackett)
   - Enable RSS Sync: ✓
   - Enable Interactive Search: ✓
   - URL: `http://jackett:9117/api/v2.0/indexers/<indexer-id>/results/torznab/`
   - API Key: (from Jackett)
   - Categories: 5000,5010,5020,5030,5040,5045,5050 (TV)
   - Test and Save

**Configure Media Management:**

1. Go to Settings → Media Management:
   - Enable "Rename Episodes": ✓
   - Root Folders: Add `/tv`
   - Episode Naming: Configure as desired
   - Permissions: Set CHMOD folder to `755`

**Series Types:**

- Standard: Regular TV shows
- Anime: Japanese animation
- Daily: News, talk shows

### Jellyseerr ([Docs](https://docs.jellyseerr.dev/))

**Initial Setup:**

1. Access at `http://<your-ip>:5056`.
2. Sign in with Jellyfin account or create a local account.
3. Complete the setup wizard.

**Configure Services:**

1. Go to Settings → Services → Radarr:

   - Default Server: ✓
   - Server Name: `Radarr`
   - Hostname or IP: `radarr`
   - Port: `7878`
   - API Key: (from Radarr Settings → General → API Key)
   - Base URL: (leave empty)
   - Quality Profile: (select default)
   - Root Folder: `/movies`
   - Test and Save

2. Go to Settings → Services → Sonarr:
   - Default Server: ✓
   - Server Name: `Sonarr`
   - Hostname or IP: `sonarr`
   - Port: `8989`
   - API Key: (from Sonarr Settings → General → API Key)
   - Base URL: (leave empty)
   - Quality Profile: (select default)
   - Root Folder: `/tv`
   - Language Profile: English (or preferred)
   - Test and Save

**Configure Jellyfin (Optional):**

1. Go to Settings → Services → Jellyfin:
   - Hostname or IP: `jellyfin`
   - Port: `8096`
   - Use SSL: No
   - Test connection and save

### Jellyfin ([Docs](https://jellyfin.org/docs/general/installation/))

**Initial Setup:**

1. Access at `http://<your-ip>:8096`.
2. Select your preferred language.
3. Create an administrator account with a strong password.
4. Complete the setup wizard.

**Configure Libraries:**

1. Go to Dashboard → Libraries → Add Media Library:

   **Movies Library:**

   - Content type: Movies
   - Display name: Movies
   - Folders: Add `/data/movies`
   - Preferred language: English (or your preference)
   - Country: (your country)
   - Enable metadata downloaders: TheMovieDb, The Open Movie Database
   - Save

   **TV Shows Library:**

   - Content type: TV Shows
   - Display name: TV Shows
   - Folders: Add `/data/tvshows`
   - Preferred language: English (or your preference)
   - Country: (your country)
   - Enable metadata downloaders: TheMovieDb, TheTVDB
   - Save

**Additional Configuration:**

1. Dashboard → Playback → Transcoding:

   - Hardware acceleration: None (or configure if you have GPU support)
   - Enable hardware decoding for: (select relevant codecs)

2. Dashboard → Users:

   - Create additional user accounts as needed
   - Set up parental controls if required

3. Dashboard → Plugins:
   - Install useful plugins like Trakt, Bookshelf, or others as needed

**Mobile Apps:**

- Download Jellyfin apps for iOS/Android
- Use server address: `http://<your-ip>:8096`

---

### Docker Networking Tips

- Use container names as hostnames for service-to-service communication (e.g., `qbittorrent`, `jackett`, `radarr`, `sonarr`).
- No need to use IP addresses within the Docker network.
- If you change service names in `docker-compose.yml`, update references accordingly.

## Step 8: Stop the Services

To stop all services, run:

```bash
docker compose down
```

## Troubleshooting

### Common Issues and Solutions

**1. VPN Connection Issues:**

```bash
# Check VPN container logs
docker logs vpn

# Check if VPN is connected
docker exec qbittorrent curl -s ifconfig.me

# If using custom VPN, ensure your .ovpn file is correct
# Check the vpn folder permissions
ls -la vpn/
```

**2. qBittorrent Issues:**

```bash
# Check qBittorrent logs
docker logs qbittorrent

# Get temporary admin password
docker logs qbittorrent | grep "WebUI admin password"

# Reset qBittorrent config (will delete settings)
docker stop qbittorrent
sudo rm -rf ./appdata/qbittorrent/config/*
docker start qbittorrent
```

**3. Permission Issues:**

```bash
# Fix ownership of appdata and media folders
sudo chown -R $USER:$USER ./appdata ./downloads ./media

# Check current user ID and group ID
id $(whoami)

# Update .env file with correct PUID and PGID
```

**4. Service Not Accessible:**

```bash
# Check if containers are running
docker ps

# Check container logs
docker logs <container_name>

# Restart specific service
docker restart <container_name>

# Check network connectivity between containers
docker exec <container_name> ping <other_container_name>
```

**5. Download Path Issues:**

```bash
# Verify folder structure exists
ls -la downloads/ media/

# Create missing folders
mkdir -p downloads/{movies,tv,incomplete,torrents}
mkdir -p media/{movies,tv}

# Check volume mounts
docker inspect <container_name> | grep -A 10 Mounts
```

**6. API Connection Issues (Radarr/Sonarr with qBittorrent/Jackett):**

- Use container names as hostnames (e.g., `qbittorrent`, `jackett`)
- Ensure all services are on the same Docker network
- Check firewall rules if running on remote server
- Verify API keys are correct

**7. Jellyfin Transcoding Issues:**

```bash
# Check Jellyfin logs
docker logs jellyfin

# For hardware acceleration, ensure proper GPU drivers
# Add to docker-compose.yml under jellyfin service:
#   deploy:
#     resources:
#       reservations:
#         devices:
#           - driver: nvidia
#             count: all
#             capabilities: [gpu]
```

### Useful Commands

- Check the logs of a specific container:

```bash
docker logs <container_name>
```

- Restart a specific container:

```bash
docker restart <container_name>
```

- Rebuild and restart all services:

```bash
docker compose down && docker compose up -d
```

- Remove all containers and start fresh (keeps data):

```bash
docker compose down
docker compose up -d
```

- Check container resource usage:

```bash
docker stats
```

- Execute commands inside a running container:

```bash
docker exec -it <container_name> /bin/bash
```

## Additional Notes

- Ensure that the `vpn.conf` file is correctly configured for your VPN provider.
- Modify the `docker-compose.yml` file if you need to customize ports or volumes.
- Check Your Public IP Address `curl https://ipv4.icanhazip.com/` . This can be useful to configure your VPN or for other networking purposes.

---

## Advanced Configuration

### Hardware Acceleration for Jellyfin

If you have a compatible GPU, you can enable hardware acceleration for Jellyfin transcoding:

**NVIDIA GPU:**

1. Install NVIDIA Container Toolkit on your host system
2. Add the following to the jellyfin service in `docker-compose.yml`:
   ```yaml
   deploy:
     resources:
       reservations:
         devices:
           - driver: nvidia
             count: all
             capabilities: [gpu]
   ```

**Intel Quick Sync:**
Add device mapping to jellyfin service:

```yaml
devices:
  - /dev/dri:/dev/dri
```

### Custom Network Configuration

If you need custom networking (not recommended for beginners):

1. Create a custom network in `docker-compose.yml`:

   ```yaml
   networks:
     media_network:
       driver: bridge
   ```

2. Comment out `network_mode: service:vpn` lines
3. Add the network to each service that needs VPN protection

### Reverse Proxy Setup

For secure external access, consider using a reverse proxy like Nginx Proxy Manager or Traefik:

1. Add labels to services for automatic discovery
2. Use SSL certificates for HTTPS
3. Configure authentication for sensitive services

### Custom Quality Profiles

**Radarr Quality Profiles:**

- **Ultra HD**: 4K content only
- **HD**: 1080p and 720p
- **SD**: Standard definition for older content

**Sonarr Quality Profiles:**

- **WEB-DL Preferred**: Prioritize web downloads over TV rips
- **Animation**: Optimized for anime content
- **Archive**: High quality for long-term storage

### Automated Backup

Create a backup script for your configuration:

```bash
#!/bin/bash
# backup-config.sh
DATE=$(date +%Y%m%d_%H%M%S)
tar -czf "backup_$DATE.tar.gz" appdata/ .env docker-compose.yml

# Keep only last 7 backups
ls -t backup_*.tar.gz | tail -n +8 | xargs rm -f
```

---

## Best Practices

### Security Recommendations

1. **Change Default Credentials**: Always change default usernames and passwords
2. **Use Strong Passwords**: Generate unique, complex passwords for each service
3. **VPN Always On**: Never disable VPN for torrent traffic
4. **Regular Updates**: Keep Docker images updated with `docker compose pull`
5. **Firewall Configuration**: Only expose necessary ports to the internet

### Performance Optimization

1. **SSD Storage**: Use SSD for Docker volumes and databases
2. **Sufficient RAM**: Allocate adequate memory for transcoding
3. **Fast Internet**: Ensure good bandwidth for downloads and streaming
4. **Regular Cleanup**: Remove old torrents and clear download folders

### Monitoring and Maintenance

1. **Log Rotation**: Configure log rotation to prevent disk space issues
2. **Health Checks**: Monitor container health with `docker ps`
3. **Disk Space**: Monitor available space for downloads and media
4. **Network Performance**: Check VPN connection speed regularly

### Content Organization

1. **Consistent Naming**: Use Radarr/Sonarr for consistent file naming
2. **Quality Settings**: Set appropriate quality profiles for your storage capacity
3. **Language Preferences**: Configure preferred languages in each service
4. **Metadata Sources**: Enable multiple metadata sources for better results

### Legal Considerations

- **Copyright Compliance**: Only download content you legally own or have permission to access
- **Local Laws**: Understand and comply with your local copyright laws
- **VPN Jurisdiction**: Choose VPN providers in privacy-friendly jurisdictions
- **Private Trackers**: Consider using private trackers for better security and quality

---

## Frequently Asked Questions

### Q: Why is my VPN not connecting?

A: Check your VPN configuration file and credentials. Ensure the OpenVPN file is correctly formatted and contains all necessary certificates.

### Q: Can I run this without VPN?

A: While technically possible by commenting out VPN-related lines, it's strongly discouraged for legal and privacy reasons.

### Q: How do I add more indexers to Jackett?

A: Access Jackett's web interface, click "Add indexer", search for your desired tracker, and configure it with your credentials.

### Q: Why are downloads slow?

A: Check your VPN connection speed, qBittorrent connection limits, and ensure you're connected to healthy torrents with good seeders.

### Q: How do I backup my configuration?

A: The `appdata` folder contains all configurations. Regularly backup this folder along with your `.env` and `docker-compose.yml` files.

### Q: Can I use this on Windows or macOS?

A: Yes, but you'll need Docker Desktop. Some VPN configurations may require adjustments for different operating systems.

### Q: How do I update the services?

A: Run `docker compose pull` to download updates, then `docker compose up -d` to restart with new images.

### Q: What if a service won't start?

A: Check the logs with `docker logs <container_name>`, verify your configuration, and ensure all required folders exist.

---
