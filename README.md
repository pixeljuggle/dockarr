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
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
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

For example, on Linux:

```bash
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

Verify the installation:

```bash
docker-compose --version
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

## Step 5: Configure VPN

Place your VPN configuration file in the `vpn` folder. For example, the file should be located at `vpn/vpn.conf`.

## Step 6: Start the Services

Run the following command to start all services:

```bash
docker-compose up -d
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

1. Access the Web UI at `http://<your-ip>:8900`.
2. A temporary password for the `admin` user will be printed to the container log on startup. You must then change username/password in the web UI (change immediately in Web UI → Settings → Web UI).
3. If you do not change the password, it will reset to default on every container start.
4. Download paths are pre-set to `/downloads/` and incomplete to `/downloads/incomplete/`.
5. You can check logs for startup info: `docker logs qbittorrent`.

### Jackett ([Docs](https://github.com/Jackett/Jackett#readme))

1. Access at `http://<your-ip>:9117`.
2. Click "Add Indexer" and select or search for your preferred trackers. Enter credentials if required.
3. Copy the API Key from the Jackett dashboard (top right) for use in Sonarr/Radarr.
4. For Sonarr/Radarr, use the Torznab feed URL: `http://jackett:9117/api/v2.0/indexers/<indexer>/results/torznab/`.

### Radarr ([Docs](https://wiki.servarr.com/radarr))

1. Access at `http://<your-ip>:7878`.
2. Go to Settings → Download Clients → Add → qBittorrent.
   - Host: `qbittorrent` (container name)
   - Port: `8900`
   - Username/Password: as set in qBittorrent
   - Category: `movies` (create in qBittorrent if needed)
3. Go to Settings → Indexers → Add → Torznab → Custom.
   - URL: `http://jackett:9117/api/v2.0/indexers/<indexer>/results/torznab/`
   - API Key: from Jackett
4. Set Root Folder to `/movies`.

### Sonarr ([Docs](https://wiki.servarr.com/sonarr))

1. Access at `http://<your-ip>:8989`.
2. Go to Settings → Download Clients → Add → qBittorrent.
   - Host: `qbittorrent`
   - Port: `8900`
   - Username/Password: as set in qBittorrent
   - Category: `tv` (create in qBittorrent if needed)
3. Go to Settings → Indexers → Add → Torznab → Custom.
   - URL: `http://jackett:9117/api/v2.0/indexers/<indexer>/results/torznab/`
   - API Key: from Jackett
4. Set Root Folder to `/tv`.

### Jellyseerr ([Docs](https://github.com/Fallenbagel/jellyseerr#readme))

1. Access at `http://<your-ip>:5056`.
2. Complete the initial setup and create an admin account.
3. Connect to Radarr and Sonarr:
   - Radarr Host: `http://radarr:7878` (API Key from Radarr Settings → General)
   - Sonarr Host: `http://sonarr:8989` (API Key from Sonarr Settings → General)
4. Optionally, connect to Jellyfin (`http://jellyfin:8096`) or Plex.

### Jellyfin ([Docs](https://jellyfin.org/docs/))

1. Access at `http://<your-ip>:8096`.
2. Complete the setup wizard and create an admin user.
3. Add libraries:
   - Movies: `/data/movies`
   - TV Shows: `/data/tvshows`
4. Configure users, metadata, and plugins as needed.

---

### Docker Networking Tips

- Use container names as hostnames for service-to-service communication (e.g., `qbittorrent`, `jackett`, `radarr`, `sonarr`).
- No need to use IP addresses within the Docker network.
- If you change service names in `docker-compose.yml`, update references accordingly.

## Step 8: Stop the Services

To stop all services, run:

```bash
docker-compose down
```

## Troubleshooting

- Check the logs of a specific container:

```bash
docker logs <container_name>
```

- Restart a specific container:

```bash
docker restart <container_name>
```

## Additional Notes

- Ensure that the `vpn.conf` file is correctly configured for your VPN provider.
- Modify the `docker-compose.yml` file if you need to customize ports or volumes.
- Check Your Public IP Address `curl https://ipv4.icanhazip.com/` . This can be useful to configure your VPN or for other networking purposes.
