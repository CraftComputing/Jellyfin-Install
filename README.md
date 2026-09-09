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
`sudo apt-get install -y apt-transport-https gnupg curl -y`\
`curl -fsSL https://repo.jellyfin.org/debian/jellyfin_team.gpg.key | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/debian-jellyfin.gpg`

### Install Jellyfin Server
`sudo apt update`\
`sudo apt install jellyfin -y`

### Set Jellyfin as a service
`sudo systemctl enable jellyfin`\
`sudo systemctl start jellyfin`

### Verify Jellyfin is running
`sudo systemctl status jellyfin`

### Check the web server
`curl -I http://localhost:8096`\
If the web server is running, if should respond with "HTTP/1.1 302 Found"

OR

Open a web browser to http://YOUR-IP:8096

### Jellyfin Basic Setup

Walk through the wizard to set up a User and Password.

If you already have media in a local directory, you can set up Library folders now. If you plan on connecting to a network share, those instructions will be covered shortly.

# Setting up Hardware Transcoding

### Intel i915/i965 GPUs (Pre ARC)

Drivers for Intel i915/i965 are part of the Linux Kernel, so no download required.

In the terminal, enter:
`sudo usermod -aG video $USER`

Install Intel GPU Tools for monitoring
`sudo apt update && sudo apt install intel-gpu-tools -y`

Reboot the system for changes to take effect
`sudo reboot`

Log back into the Jellyfin Webapp, and navigate to:
`Dashboard -> Playback -> Streaming -> Transcode`

Under 'Hardware acceleration' pulldown menu, select:
`Intel Quicksync (QSV`

Enable hardware decoding for: H.264, HEVC (Main10), HEVC, MPEG-2, AV1
Enable hardware encoding for: H.264, HEVC

Click "Save" at the bottom of the screen.

Reboot Jellyfin for changes to take effect
`sudo systemctl restart jellyfin`

To monitor transcoding, in the terminal, enter:
`intel_gpu_top`


 ### Intel ARC GPUs

Verify Intel drivers are installed and up to date:
`sudo apt update && sudo apt install intel-media-driver vainfo intel-ucode -y`

Intel i915 support may need to be added to the OS.
`sudo nano /etc/default/grub`\
Add the following line:
`i915.enable_guc=3`

Save changes and exit nano.

In the terminal, enter:
`sudo usermod -aG video $USER`

Update GRUB, and reboot the system:
`sudo update-grub`\
`sudo reboot`

Log back into the Jellyfin Webapp, and navigate to:
`Dashboard -> Playback -> Streaming -> Transcode`

Under 'Hardware acceleration' pulldown menu, select:
`Intel Quicksync (QSV`

Enable hardware decoding for: H.264, HEVC, VP9, AV1
Enable hardware encoding for: H.264, HEVC, AV1

Reboot Jellyfin for changes to take effect
`sudo systemctl restart jellyfin`


### AMD GPUs (VAAPI)

Verify AMD GPU drivers are installed and up to date:
`sudo apt update && sudo apt install mesa-va-drivers va-driver-all vainfo -y`

In the terminal, enter:
`sudo usermod -aG video $USER`

Reboot the system for changes to take effect
`sudo reboot`

Log back into the Jellyfin Webapp, and navigate to:
`Dashboard -> Playback -> Streaming -> Transcode`

Under 'Hardware acceleration' pulldown menu, select:
`AMD AMF`

Enable hardware decoding for: H.264, HEVC, VP9, AV1 (AV1 decode only available on RDNA2 and newer)
Enable hardware encoding for: H.264, HEVC, AV1 (AV1 encode only available on RDNA3 and newer)

Reboot Jellyfin for changes to take effect
`sudo systemctl restart jellyfin`


### NVIDIA GPUs

Install NVIDIA Proprietary Drivers
`sudo apt update && sudo ubuntu-drivers autoinstall`

Reboot server after installation
`sudo reboot`

Install Jellyfin NVIDIA-specific support
`sudo apt install jellyfin-nvidia -y`

Allow Jellyfin and Nvidia access to video hardware
`sudo usermod -aG video,jellyfin,nvidia jellyfin`

Reboot the server after installation
`sudo reboot`

Log back into the Jellyfin Webapp, and navigate to:
`Dashboard -> Playback -> Streaming -> Transcode`

Under 'Hardware acceleration' pulldown menu, select:
`Nvidia NVENC`


# Network Storage for Media

If using another system or NAS for storing Jellyfin media, use the following steps to mount a network path.

Install CIFS Utilities
`sudo apt update && sudo apt install cifs-utils -y`

Create a mount point
`sudo mkdir -p /mnt/jellyfin

Create a credential file, containing username and password for your network storage
`sudo nano /root/.smbcredentials`\
\
`username=your_nas_username`\
`password=your_nas_password`\
`domain=YOUR_WORKGROUP`

Lock permissions to the smbcredential file:
`sudo chmod 600 /root/.smbcredentials`

Add network path to fstab for automatic mounting
`sudo nano /etc/fstab`\
\
Add the following line to the bottom of the file:
`//YOUR_NAS_IP/media_share /mnt/jellyfin cifs credentials=/root/.smbcredentials,iocharset=utf8,file_mode=0777,dir_mode=0777 0 0`

Update and reload systemd manager and configuration:
`sudo systemctl daemon-reload`

Mount the network directory
(This happens automatically at boot, but we can load it manually with the following command)
`sudo mount -a`

Verify your directory loaded by checking directory content:
`cd /mnt/jellyfin`\
`ls -a`
