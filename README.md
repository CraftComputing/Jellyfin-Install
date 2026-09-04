# Jellyfin-Install
Installing Jellyfin on Ubuntu/Debian

### Pre-Install
This tutorial was written for Ubuntu 24.04 Server/Desktop, and should work on most Debian-based OSes.

Install Ubuntu 24.04

Run
`sudo apt update && sudo apt upgrade`

Reboot to apply updates.

# Installing Jellyfin (Native)
We will install Jellyfin on bare metal, rather than in a Docker container.

### Add the Jellyfin Repository
`echo "deb https://repo.jellyfin.org/ubuntu $(lsb_release -c -s) main" | sudo tee /etc/apt/sources.list.d/jellyfin.list`

### Add the Jellyfin GPG key
`sudo apt-get install -y apt-transport-https gnupg-curl`
`sudo curl https://repo.jellyfin.org/ubuntu/jellyfin_team.gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/jellyfin.gpg`

### Install Jellyfin Server
`sudo apt update`
`sudo apt install jellyfin -y`

### Set Jellyfin as a service
`sudo systemctl enable jellyfin`
`sudo systemctl start jellyfin`

### Verify Jellyfin is running
`sudo systemctl status jellyfin`

### Check the web server
`curl -I http://localhost:8096`
If the web server is running, if should respond with "HTTP/1.1 302 Found"

OR

Open a web browser to http://YOUR-IP:8096

# Setting up Jellyfin

