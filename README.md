# Homescripts README

This is my configuration that works for me the best for my use case of an all in one media server. This is not meant to be a tutorial by any means as it does require some knowledge to get setup. I'm happy to help out as best as I can and welcome any updates/fixes/pulls to make it better and more helpful to others.

I use the latest rclone stable version downloaded direclty via the [script install](https://rclone.org/install/#script-installation) as package managers are frequently out of date and not maintained.

[Change Log](https://github.com/animosity22/homescripts/blob/master/Changes.MD)
## Home Configuration

- Verizon Gigabit Fios
- Dropbox with encrypted media folder
- Linux
- Intel(R) Core(TM) i7-7700 CPU @ 3.60GHz
- 32 GB of Memory
- 1TB - root/system disk
- 2TB SSD for rclone disk cache
- Spinning disk for big storage pools

I adjusted my mounts on my Linux machine to use BTRFS over EXT4/XFS and found a huge performance improvement.

/etc/fstab
```
/dev/disk/by-uuid/f12cd4cf-d9e5-4022-8d16-5ccde5c4273e / btrfs defaults 0 1
/dev/disk/by-uuid/7B20-481C /boot/efi vfat defaults 0 1

# SSD
UUID=7f93b2af-ad87-4db1-aa82-682136cec07a /cachet auto defaults 0 0
UUID=e065dcaa-e548-45e6-a226-f5ea83b5ab22 /data auto defaults 0 0
UUID=b6e7996a-01cc-4776-9f19-4cfb7dc46b7a /seed auto defaults 0 0
```

## Dropbox
I migrated away from Google Drive to Dropbox as there still is an Enterprise Standard plan that seems to be unlimited space but I disliked
the upload and download limits so I made the change. Dropbox is similiar to API usage compared to Google but there is not a pacer by default
I work around that by setting a limit for transactions per second on the API via my mount command. API usage in Dropbox is tied to each
application registration so I seperate out my apps and use one for uploading, one for movies and one for television shows as to never
overload a particular one and allow easy reporting in the console.

## My Workflow

I use Sonarr and Radarr in conjuction with NZBGet and qBittorrent to get my media.

My normal work flow grabs a file, downloads it to spinning disk (/seed), copies over to the proper /media folder and uploads automatically after an hour based on the rclone mount settings.
The goal here was to remove mergerfs from my workflow as it is another layer and I wanted to reduce complexity that did not give me value. 

### Installation
My Linux setup:

```
PRETTY_NAME="Ubuntu 22.04 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04 (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```

Fuse needs to be installed for a rclone mount to function. `allow-other` is for my use and not recommended for shared servers as it allows any user to see a rclone mount. I am on a dedicated server that only I use so that is why I uncomment it.


You need to make the change to /etc/fuse.conf to `allow_other` by uncommenting the last line or removing the # from the last line.

	sudo vi /etc/fuse.conf
	root@gemini:~# cat /etc/fuse.conf
	# /etc/fuse.conf - Configuration file for Filesystem in Userspace (FUSE)
	
	# Set the maximum number of FUSE mounts allowed to non-root users.
	# The default is 1000.
	#mount_max = 1000

	# Allow non-root users to specify the allow_other or allow_root mount options.
	user_allow_other
	

```
/media
	/Movies (rclone mount with vfs cache mode full)
	/TV (rclone mount with vfs cache mode full)
```

My `rclone.conf` has an entry for the Google Drive connection and and encrypted folder in my Google Drive called `media`. I mount media with a rclone script to display the decrypted contents on my server. 

My rclone looks like: [rclone.conf](https://github.com/animosity22/homescripts/blob/master/rclone.conf)

They are all mounted via systemd scripts. rclone is mounted first followed by the mergerfs mount.

My media starts up items in order:
1) [rclone-movies service](https://github.com/animosity22/homescripts/blob/master/systemd/rclone-movies.service) This is a standard rclone mount, the post execution command allows for the caching of the file structure in a single systemd file that simplies the process.

2) [rclone-tv service](https://github.com/animosity22/homescripts/blob/master/systemd/rclone-tv.service) This is a standard rclone mount, the post execution command allows for the caching of the file structure in a single systemd file that simplies the process.
### Docker
I recently made the switch to containerize everything and move to dockers for ease of use mainly with Plex. For using hardware decoding
for HDR tone maps, it's cumbersome to get working on any other OS than Ubuntu and docker provided by Plex already does this.
I use an override.conf for my docker startup and require rclone mounts to be active or my dockers shutdown. This is to prevent any issues with mounts being empty or the order of boot as
I want my rclone mounts to be ready and running for dockers as that's how I had my systemd services prior.

```
gemini:/etc/systemd/system/docker.service.d # cat override.conf
[Unit]
After=rclone-movies.service rclone-tv.service
Requires=rclone-movies.service rclone-tv.service
```

I use docker compose for all my serivces and have portainer there for easier looking at things when I don't want to connect to a console. I use the same user ID/groups for my docker to
simpify permissions. My plex compose is basic and looks like:

```
  plex:
    image: lscr.io/linuxserver/plex
    container_name: plex
    network_mode: host
    devices:
     - /dev/dri:/dev/dri
    privileged: true
    environment:
      - PUID=1000
      - PGID=1000
      - VERSION=docker
      - TZ=America/New_York
    volumes:
      - /opt/docker/data/plex:/config
      - /media/Movies:/media/Movies
      - /media/TV:/media/TV
    restart: unless-stopped
	```

	/dev/dri is a must for hardware transocding.

## Plex Tweaks```
I used to use an override for my plexmediaserver service to get around it running as the plex user and require services be running. This allows me to keep my trash empty on as if mount
has a problem, it will stop plex.

```
gemini: /etc/systemd/system/plexmediaserver.service.d # cat override.conf
[Unit]
After=rclone-movies.service rclone-tv.service
Requires=rclone-movies.service rclone-tv.service

[Service]
User=felix
Group=felix
```

These tips and more for Linux can be found at the [Plex Forum Linux Tips](https://forums.plex.tv/t/linux-tips/276247).
### Plex
- `Enable Thumbnail previews` - off: This creates a full read of the file to generate the preview and is set per library that is setup
- `Perform extensive media analysis during maintenance` - off: This is listed under Scheduled Tasks and does a full download of files and is ony used for bandwidth analysis when streaming.

### Sonarr/Radarr
- `Analyze video files` - off: This also fully downloads files to perform analysis and should be turned off as this happens frequently on library refreshes if left on.

## Caddy Proxy Server

I use Caddy to server majority of my things as I plug directly into GitHub oAuth2 for authentication. I can toggle CDN on and off via the proxy in the DNS.

My configuration is [here](https://github.com/animosity22/homescripts/blob/master/PROXY.MD).
