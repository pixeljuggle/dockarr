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
