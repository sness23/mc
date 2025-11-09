# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a **Paper Minecraft Server** installation running version 1.21.8 (MC: 1.21.8). Paper is a high-performance fork of Spigot/Bukkit that provides additional optimizations and features for Minecraft multiplayer servers.

## Running the Server

### Starting the Server
```bash
# Basic start
java -jar paper-1.21.8-60.jar --nogui

# Recommended with memory allocation (2GB min/max)
java -Xms2G -Xmx2G -jar paper-1.21.8-60.jar --nogui
```

The server requires Java 21+ (currently using OpenJDK 21.0.8).

**Memory allocation flags:**
- `-Xms` - Initial heap size
- `-Xmx` - Maximum heap size
- Recommended: Set both to the same value to prevent heap resizing overhead

### Server Management
- **Stop the server**: Use the `/stop` command in the console
- **Logs**: Located in `logs/` directory (latest.log for current session)
- **World data**: Main world in `world/`, Nether in `world_nether/`, End in `world_the_end/`

## Configuration Architecture

Paper uses a multi-layered configuration system with clear hierarchy:

### Core Configuration Files
1. **server.properties** - Base Minecraft server settings (port, gamemode, difficulty, max players, etc.)
2. **bukkit.yml** - Bukkit/Paper base configuration (spawn limits, chunk GC, tick rates)
3. **spigot.yml** - Spigot-specific settings (entity activation ranges, growth rates, mob behavior)
4. **config/paper-global.yml** - Paper global configuration (chunk loading, anticheat, packet limiting, watchdog)
5. **config/paper-world-defaults.yml** - Default world-specific Paper settings
6. **world/paper-world.yml** - Per-world overrides for Paper configuration

### Configuration Precedence
- World-specific settings override global defaults
- Paper configs override Spigot configs, which override Bukkit configs
- server.properties is the foundation layer

### Plugin Configuration
- **plugins/spark/config.json** - Spark performance profiler settings
- **plugins/bStats/config.yml** - bStats metrics collection

## Plugins

### Installed Plugins
- **spark** - Performance profiling and monitoring plugin (bundled with Paper)
  - Enabled in `config/paper-global.yml` under `spark.enabled: true`
  - Background profiler enabled via `plugins/spark/config.json`
  - Documentation: https://spark.lucko.me/docs/

- **ViaVersion** (5.5.1) - Protocol translation to support multiple Minecraft client versions
  - Allows newer server to support older Java Edition clients
  - Download: https://ci.viaversion.com/
  - Config: Auto-generated in `plugins/ViaVersion/`

- **ViaBackwards** (5.5.1) - Allows older clients to connect
  - Requires ViaVersion 5.5.1+ (same or newer version)
  - Download: https://ci.viaversion.com/

- **Geyser-Spigot** (2.9.0-SNAPSHOT) - Bedrock Edition cross-play support
  - Allows Minecraft Bedrock clients (mobile, console, Windows 10) to connect to Java Edition server
  - Default port: 19132 (UDP)
  - Download: https://geysermc.org/download
  - Config: `plugins/Geyser-Spigot/config.yml`
  - **Note:** Requires unique port. If "Address already in use" error occurs, check if another instance is running or change port in config

- **bStats** - Plugin metrics collection

### Updating Plugins
1. Download new plugin JAR from official source
2. Stop the server
3. Replace old JAR in `plugins/` directory
4. Start the server

**Version compatibility:** ViaBackwards requires ViaVersion of the same or newer version. Always update ViaVersion first.

### Plugin Development
- Plugins go in `plugins/` directory
- Paper uses a remapping system (see `plugins/.paper-remapped/`)
- Plugin data typically stored in subdirectories under `plugins/`

## Key Configuration Sections

### Performance Tuning
Located across multiple files:
- **Chunk loading**: `config/paper-global.yml` (chunk-loading-basic, chunk-loading-advanced, chunk-system)
- **Entity optimization**: `spigot.yml` (entity-activation-range, entity-tracking-range)
- **View/simulation distance**: `server.properties` (view-distance, simulation-distance) and `spigot.yml` per-world overrides
- **Mob spawn limits**: `bukkit.yml` (spawn-limits section)

### Network & Security
- **Server port**: `server.properties` (default: 25565)
- **Packet limiting**: `config/paper-global.yml` (packet-limiter section)
- **Proxy support**: `config/paper-global.yml` (proxies section for BungeeCord/Velocity)
- **Connection throttling**: `bukkit.yml` (settings.connection-throttle)

### World Settings
- **World generator**: `server.properties` (level-type, generator-settings)
- **Structure seeds**: `spigot.yml` (seed-village, seed-desert, etc.)
- **Game rules**: Stored in `world/level.dat` (use `/gamerule` command)

## Important Directories

- **world/**, **world_nether/**, **world_the_end/** - World data (region files, playerdata, entities)
- **plugins/** - Plugin JARs and data
- **libraries/** - Paper and plugin dependencies
- **logs/** - Server logs
- **cache/** - Server cache files
- **versions/** - Paper version data

## Updating the Server

### Update Paper
```bash
# Download latest from https://papermc.io/downloads/paper
# Replace the JAR and restart with new filename
java -Xms2G -Xmx2G -jar paper-1.21.8-XX.jar --nogui
```

### Check for Updates
- Paper update check happens on startup (watch for warnings)
- Download from: https://papermc.io/downloads/paper
- Version history tracked in `version_history.json`

## Troubleshooting

### Geyser "Address already in use" Error
If Geyser fails to start with `bind(..) failed: Address already in use`:
1. Check if another server instance is running: `lsof -i :19132` or `netstat -tulpn | grep 19132`
2. Stop any conflicting processes
3. Or change Geyser's port in `plugins/Geyser-Spigot/config.yml` (look for `bedrock.port: 19132`)

### Plugin Version Mismatches
- ViaBackwards requires matching or newer ViaVersion (both must be 5.5.1+)
- Always update ViaVersion first, then ViaBackwards
- Check plugin logs during startup for compatibility warnings

### Performance Issues
- Monitor with spark profiler: `/spark profiler start`
- Check chunk loading settings in `config/paper-global.yml`
- Adjust entity activation ranges in `spigot.yml`
- Review view/simulation distance in `server.properties`

## Documentation References

- Paper: https://docs.papermc.io/
- Bukkit config: https://docs.papermc.io/paper/reference/bukkit-configuration/
- Spigot config: https://docs.papermc.io/paper/reference/spigot-configuration/
- Paper global config: https://docs.papermc.io/paper/reference/global-configuration/
- Paper world config: https://docs.papermc.io/paper/reference/world-configuration/
- Discord: https://discord.gg/papermc

## Server Metadata Files

- **banned-ips.json**, **banned-players.json** - Ban lists
- **ops.json** - Server operators
- **whitelist.json** - Whitelist (if enabled)
- **usercache.json** - Player UUID cache
- **permissions.yml** - Permission configuration
- **commands.yml** - Command aliases and overrides
- **eula.txt** - Minecraft EULA acceptance (must be `eula=true`)
