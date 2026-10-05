# Servarr Media Stack with Soulseek and ROMM

Docker Compose stack for the Servarr applications, Jellyfin, Soulseek, and ROMM

## Requirements

- Docker Engine with the Compose plugin
- Linux or a NAS running Docker; Windows requires Docker Desktop or WSL2 path integration
- A VPN provider supported by Gluetun for qBittorrent and slskd
- Host directories for configuration and media with permissions matching `PUID` and `PGID`

Native Linux or NAS paths are the supported default. The stack also requests `/dev/net/tun` for the VPN and `/dev/dri` for Jellyfin hardware transcoding. Remove or adjust those device mappings when the host does not provide them

## Install

1. Install [Docker](https://docs.docker.com/get-docker/)
2. Clone this repo to your local machine

    ```bash
    git clone https://github.com/posterplant/servarr-media-stack.git
    ```

    or download the zip and extract it to your local machine

3. Copy `.env.example` to `.env` and replace every `change-me` value. Add your VPN credentials and API keys as needed
4. Create the host directories from `FOLDER_FOR_CONFIGS` and `FOLDER_FOR_MEDIA`. The Compose file creates container mounts, but it cannot fix host ownership or permissions
5. I've provided example configuration files for slskd and Soularr. Copy the example files from `exampleConfigs/` into the matching directories under `FOLDER_FOR_CONFIGS`:

    ```text
    FOLDER_FOR_CONFIGS/slskd/slskd.yml
    FOLDER_FOR_CONFIGS/soularr/config.ini
    ```

   Replace the credentials and API keys in both files before starting the services
6. Validate the configuration from the repository root:

    ```bash
    docker compose config --quiet
    ```

7. Start the stack:

    ```bash
    docker compose up -d
    ```

8. Configure the applications using the ports listed in `.env`, then connect Lidarr, Sonarr, and Radarr to your indexer and download clients

## Services

All published ports are configurable in `.env` and bind to every host interface by default.

| Service | Purpose | Default port |
| --- | --- | ---: |
| qBittorrent | Torrent downloads, routed through Gluetun | 8200 |
| slskd | Soulseek client, routed through Gluetun | 5030 |
| SABnzbd | Usenet downloads | 8100 |
| Prowlarr | Indexer management | 9696 |
| Radarr | Movies | 7878 |
| Sonarr | Series | 8989 |
| Lidarr | Music | 8686 |
| Bazarr | Subtitles | 6767 |
| Jellyfin | Media playback | 8097 |
| Jellyseerr | Media requests | 5055 |
| Soularr | Soulseek-to-Lidarr automation | 8265 |
| ROMM | ROM library | 1234 |
| FlareSolverr | Cloudflare proxy for supported indexers | 8191 |

Recommended initial order is: Gluetun, download clients, Prowlarr, the Arr applications, Jellyfin, then Soularr and Jellyseerr. ROMM can be configured independently

## Data Layout

The stack expects one configuration root and one media root. A typical layout is:

```text
FOLDER_FOR_CONFIGS/
    gluetun/ qbittorrent/ slskd/ sabnzbd/ bazarr/ jellyfin/
    jellyseerr/ lidarr/ prowlarr/ radarr/ sonarr/ soularr/ romm/
FOLDER_FOR_MEDIA/
    data/ media/ roms/
```

Keep downloads and library paths under the same media root so the Arr applications can import files without crossing filesystem boundaries

## Config

- Start off with the gluetun configuration. The stack will not start without a valid VPN provider and credentials. The rest of the services can be configured after the stack is running
- fill in the `.env` file with your preferred ports, PUID, PGID, and other settings. The default values are provided as examples
- I've provided example configuration files for slskd and Soularr. Copy the example files from `exampleConfigs/` into the matching directories under `FOLDER_FOR_CONFIGS`:

    ```text
    FOLDER_FOR_CONFIGS/slskd/slskd.yml
    FOLDER_FOR_CONFIGS/soularr/config.ini
    ```

   Replace the credentials and API keys in both files before starting the services
- Configure qBittorrent with your preferred settings, including download paths and connection limits. Make sure to set the download path to a directory under `/data/` so that the Arr applications can access the downloaded files
- add a balcklist of executables to the qbittorrent configuration to prevent the download of potentially harmful files
- Configure Prowlarr with your indexers
- Configure the Arr applications to use Prowlarr for indexers and the appropriate download client
- Configure Jellyfin with your media libraries and enable hardware transcoding if your host supports it
- Configure Soularr with your Soulseek credentials and set up the automation rules for importing music
- Configure ROMM with your ROM library paths and any additional settings you require
- Configure Bazarr with your subtitle preferences and link it to your media libraries
- Configure Jellyseerr with your media request settings and link it to your media libraries
