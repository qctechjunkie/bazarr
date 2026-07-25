# Radarr Docker Stack

Movie collection manager that automatically searches, downloads, and organizes your movie library.

## Quick Start

```bash
cp .env.example .env
# Edit .env to match your system
docker-compose up -d
```

Access the web interface at: http://localhost:7878

## What is Radarr?

Radarr is a movie collection manager for Usenet and BitTorrent users. It can:
- **Monitor** multiple RSS feeds for movies and grab, sort, and rename them
- **Integrate** with popular download clients like qBittorrent, Deluge, etc.
- **Automatically upgrade** quality when better versions become available
- **Rename and organize** downloaded files using customizable naming schemes

## Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PUID` | 1000 | User ID for file permissions |
| `PGID` | 1000 | Group ID for file permissions |
| `TZ` | America/New_York | Timezone |
| `RADARR_PORT` | 7878 | Web interface port |

### Directory Structure

```
radarr/
├── config/      # Radarr configuration and database (auto-created)
├── movies/      # Movie library (auto-created)
└── downloads/   # Download client output (auto-created)
```

**Important**: The `downloads` directory should be the same path used by your download client.

## Initial Setup

### 1. Access Web Interface
Navigate to http://localhost:7878 and complete the setup wizard.

### 2. Configure Download Client
Go to **Settings → Download Clients** and add your download client:

**For qBittorrent:**
- **Name**: qBittorrent
- **Host**: `qbittorrent` (if using Docker networks) or `localhost`
- **Port**: `9865`
- **Username/Password**: As configured in qBittorrent
- **Category**: `radarr` (optional but recommended)

### 3. Configure Indexers
Go to **Settings → Indexers** and add indexers manually, or use Prowlarr for automatic sync.

**Manual Configuration:**
- Add public/private torrent trackers
- Add usenet indexers
- Configure API keys and authentication

**Prowlarr Integration:**
- Indexers will sync automatically from Prowlarr
- No manual configuration needed

### 4. Configure Media Management
Go to **Settings → Media Management**:
- **Movie Naming**: Configure your preferred naming scheme
- **Folders**: Enable movie folders for better organization
- **Importing**: Configure how Radarr handles completed downloads

### 5. Add Root Folder
Go to **Settings → Media Management → Root Folders**:
- Add `/movies` as your root folder

## Integration with Other Services

### qBittorrent Setup
1. In qBittorrent, create a category called `radarr`
2. Set automatic category assignment in Radarr download client settings
3. Ensure download path is accessible to both containers

### Prowlarr Integration
1. In Prowlarr, add Radarr as an application
2. Use Radarr's API key (found in Settings → General)
3. Indexers will sync automatically

### Plex/Jellyfin/Emby Integration
- Point your media server to the same `/movies` directory
- Radarr will automatically trigger library updates after import

## Quality Profiles

Create custom quality profiles in **Settings → Profiles → Quality Profiles**:
- **HD-1080p**: For high-quality movies
- **HD-720p**: For smaller file sizes
- **4K-UHD**: For ultra-high definition (requires powerful hardware)

## Custom Formats

Use Custom Formats in **Settings → Custom Formats** to prefer/avoid specific releases:
- **Scene/P2P Groups**: Prefer trusted release groups
- **HDR/DV**: Target specific HDR formats
- **Audio Formats**: Prefer specific audio codecs

## Troubleshooting

### Movies Not Downloading
- Check **Activity** tab for search results
- Verify indexer connectivity in **Settings → Indexers**
- Check download client connection in **Settings → Download Clients**
- Review **System → Logs** for errors

### Files Not Importing
- Verify download client category matches Radarr settings
- Check file permissions (PUID/PGID)
- Ensure download and movie paths are accessible
- Review **Activity → Manual Import** for stuck files

### Quality Not Upgrading
- Check quality profile allows upgrades
- Verify **Cutoff** is set higher than current quality
- Enable **Upgrade Until Custom Format Score** if using custom formats

### Connection Issues with Other Containers
- Verify containers are on same Docker network
- Use container names for inter-container communication
- Check firewall settings
- Test connectivity: `docker exec radarr ping qbittorrent`

## Security Notes

- Secure your API key and don't share it publicly
- Use strong authentication for private indexers
- Keep Radarr updated for security patches
- Consider using a VPN for additional privacy

## Backup

Important directories to backup:
- `./config` - Contains all Radarr settings, database, and metadata

```bash
tar -czf radarr-backup-$(date +%Y%m%d).tar.gz config/
```

## Performance Tips

- Use SSD storage for Radarr database and temporary processing
- Configure appropriate retention policies for history
- Use quality profiles to limit unnecessary downloads
- Monitor disk space usage regularly

## Updates

```bash
docker-compose pull
docker-compose up -d
docker image prune
```

## Common File Naming Schemes

**Simple**: `{Movie Title} ({Release Year}) {Quality Full}`
- Example: `The Matrix (1999) Bluray-1080p.mkv`

**Detailed**: `{Movie CleanTitle} ({Release Year}) [{Quality Title}] [{MediaInfo VideoDynamicRange}] [{Mediainfo AudioCodec}-{MediaInfo AudioChannels}]`
- Example: `The Matrix (1999) [Bluray-1080p] [HDR] [DTS-7.1].mkv`