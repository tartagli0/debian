# Background
This guide provides details for configuring a general purpose home server running Debian 11.

## Hardware
I removed the broken screen from an old laptop and repurposed it as a (literally) [headless server](https://en.wikipedia.org/wiki/Headless_computer). The technical specs are:
* [Toshiba Satellite P55-A5312](https://support.dynabook.com/support/modelHome?freeText=1200006797)
* [Intel Core i5-4200U @ 1.60 GHz](https://www.intel.com/content/www/us/en/products/sku/75459/intel-core-i54200u-processor-3m-cache-up-to-2-60-ghz/specifications.html)
* 6GB DDR3L 1600MHz memory
* [PNY CS1311 2.5'' 120GB SATA III SSD](https://www.pny.com/SSD-CS1311?sku=SSD7CS1311-120-RB)
* WD My Passport 0820 2TB USB 3.0 Portable HDD

## Software
I installed [Debian 11 "Bullseye"](https://wiki.debian.org/DebianBullseye), obtained via the project's [bittorrent download page](https://cdimage.debian.org/debian-cd/current-live/amd64/bt-hybrid/). The most current stable release was [version 11.4](https://www.debian.org/News/2022/20220709).

# Transmission
The [Transmission](https://transmissionbt.com/) bittorrent client provides a daemon that facilitates downloading large files directly to a server via a web interface.

1. Install Transmission.
```bash
apt install transmission-cli transmission-common transmission-daemon
```
2. Stop running daemon.
```bash
service transmission-daemon stop
```
3. Edit the configuration file `/etc/transmission-daemon/settings.json`.
    1. Set the download directory (in this case `/data/torrents`).
    ```bash
    "download-dir": "/data/torrents",
    ```
    2. Set user name and password for web interface (both are set to *transmission* in example below).
    ```bash
    "rpc-password": "transmission",
    "rpc-username": "transmission",
    ```
    3. Allow computers on local network to access web interface (in this case, router assigns local IPs using the pattern *192.168.4.xxx*).
    ```bash
    "rpc-whitelist": "127.0.0.1,192.168.4.*",
    ```
4. Give ownership of torrent download directory to user *debian-transmission* (automatically created during Transmission installation).
```bash
chown debian-transmission:debian-transmission /data/torrents
```
5. Start daemon.
```bash
service transmission-daemon start
```

Transmission web interface should now be accessible via port 9091 using the server's IP address. For example `http://192.168.4.120:9091/transmission/web/`


# Plex Media Server
The [official installation instructions](https://support.plex.tv/articles/235974187-enable-repository-updating-for-supported-linux-server-distributions/) for adding the Plex repository on Debian and Ubuntu use the [deprecated **apt-key** utility](https://manpages.debian.org/bullseye/apt/apt-key.8.en.html). Use the steps below in lieu of section **DEB-based distros (Ubuntu, etc.)** in the official documentation.
1. Download the Plex repository's signed GPG key.
```bash
curl -o /etc/apt/trusted.gpg.d/plexmediaserver.asc \
  https://downloads.plex.tv/plex-keys/PlexSign.key
```
2. Add the Plex repository as an APT data source. See man pages for [apt-key](https://manpages.debian.org/bullseye/apt/apt-key.8.en.html) and [sources.list](https://manpages.debian.org/bullseye/apt/sources.list.5.en.html) for details.
    1. Create the file `/etc/apt/sources.list.d/plexmediaserver.sources`.
    2. Add the lines:
    ```bash
    Types: deb
    URIs: https://downloads.plex.tv/repo/deb
    Suites: public
    Components: main
    Signed-By: /etc/apt/trusted.gpg.d/plexmediaserver.asc
    ```
3. Install Plex Media Server.
```bash
apt update
apt install plexmediaserver
```

The Plex Web App can now be accessed via the server's IP address. For example: `http://192.168.4.120:32400/web`. See the [official documentation](https://support.plex.tv/articles/200288666-opening-plex-web-app/) for details.