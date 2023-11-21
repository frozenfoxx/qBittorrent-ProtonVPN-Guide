#WIP

# DockerqBitProtonVPN
Guide for setting up qBittorrent to use ProtonVPN using Docker:

In this guide we'll create two containers, one for qBittorrent from the linuxserver.io<br>
The other one is for gluetun which is what we'll use for ProtonVPN.<br>
Essentially, we're going to route qBittorrent traffic into the gluetun container.

First you'll need to login to https://account.protonvpn.com

Then you'll need to take note of your OpenVPN Credentials in here:

![image](https://github.com/Chillsmeit/DockerqBitProtonVPN/assets/93094077/cbf3ed2b-3a23-4034-bfdc-636ded533255)

I usually create a docker folder in my Home folder and create individual folders for each container, keeps things tidy.

Create the necessary folders:
```
mkdir -p "$HOME/Docker/protonvpn" && mkdir -p "$HOME/Docker/qbittorrent
```
Create the YML for gluetun:
```
touch "$HOME/Docker/protonvpn/docker-compose.yml"
```
Use cat to quickly fill in the info in the YML file
```
cat <<EOF > "$HOME/Docker/protonvpn/docker-compose.yml"
version: "3"
services:
  gluetun:
    image: qmcgaw/gluetun
    container_name: protonvpn # I like to change the container name to be the VPN I use
    cap_add:
      - NET_ADMIN
    ports: # These are the qBittorrent ports, I like to use random ports and not default ports
      - 13750:13750 # This is for the qBittorrent WebUI Port
      - 7830:7830 # Listening port for TCP
      - 7830:7830/udp # Listening port for UDP
    environment:
      - VPN_SERVICE_PROVIDER=protonvpn
      - OPENVPN_USER=username # REPLACE these with your OpenVPN credentials
      - OPENVPN_PASSWORD=password # REPLACE these with your OpenVPN credentials
      - SERVER_COUNTRIES=Netherlands
    volumes:
      - ./ProtonVPN:/gluetun
    restart: unless-stopped
EOF
```
Create the YML for qBittorrent:
```
cat <<EOF > "$HOME/Docker/qbittorrent/docker-compose.yml"
version: "2.1"
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=1000 # to find your current ID just type "id" in the terminal
      - PGID=1000 # to find your current group ID just type "id" in the terminal
      - TZ=Etc/UTC
      - WEBUI_PORT=13750 # This needs to be the exact same port we used on glueton for the WebUI
    volumes:
      - ./config:/config # this will create the config folder in the same folder as we have the yml file
      - /path/to/your/drive:/downloads # change the left part of : to your actual path where you want to store your downloads
    network_mode: "container:protonvpn" # this needs to be the exact same name as the protonvpn container we defined
    restart: unless-stopped
EOF
```
