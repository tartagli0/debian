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

## File naming conventions
Plex can be fastidious in matching file names to corresponding titles in a movie database.

For example, the movie [Blade Runner 2049](https://www.warnerbros.com/movies/blade-runner-2049) will be ignored if you name the file `Blade_Runner_2049.mkv`, as Plex will try to match a movie of the same name made in 2049. Naming the file `Blade_Runner_2049_2017.mkv` will work, as Plex will will look for the movie with that title made in 2017.

See [Naming and organizing your Movie files](https://support.plex.tv/articles/naming-and-organizing-your-movie-media-files/) in the official documentation for details.


# Samba
Install the [Samba](https://www.samba.org/samba/) server with the command:
```bash
apt install samba
```

The [server daemon](https://www.samba.org/samba/docs/current/man-html/smbd.8.html) is automatically enabled and started by [systemd](https://wiki.debian.org/systemd) when Samba is installed. Stop the running service via the [systemctl](https://manpages.debian.org/bullseye/systemd/systemctl.1.en.html) utility:
```bash
systemctl stop smbd
```

The configuration file used by the Samba server daemon is located at `/etc/samba/smb.conf` (see [Debian wiki](https://wiki.debian.org/Samba/ServerSimple) for a short example). Another copy of the default configuration is saved to `/usr/share/samba/smb.conf`, which you can use to restore initial settings.

The following sections describe the settings I used for my home server, but might not be required for every case. Edits were performed relative to the default `smb.conf` file in Debian.

**To-do:** Add explanations for all lines in config file.

## **global** section
The **global** section will apply settings to all Samba shares. Edit the default configuration by adding or commenting out lines to match the example below.
```conf
[global]
    logging = file
    log file = /var/log/samba/log.%m
    # max log size = 1000
    # panic action = /usr/share/samba/panic-action %d
    server role = standalone server
    obey pam restrictions = yes
    unix password sync = yes
    passwd program = /usr/bin/passwd %u
    passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
    pam password change = yes
    # map to guest = bad user
    # usershare allow guests = yes
```

## **homes** section
This section controls how Debian user home directories are shared via Samba.
```
[homes]
   # comment = Home Directories
   browseable = no
   # read only = yes
   create mask = 0700
   directory mask = 0700
   valid users = %S
```

## Printers
Sections **printers** and **print$** can be fully commented out, as this server configuration doesn't include any shared printers.

# To Do
1. **mailx** for notifications (e.g., Samba crash, HDD failure)
2. Add **panic action** parameter to **global** section (requires **mailx**)
3.  **smartmontools**
4. Convert README to Github Pages site