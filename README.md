# Streamlist - self-hosted music streaming server written in Go

![Screencast](https://raw.githubusercontent.com/streamlist/streamlist/master/screencast1.gif?updated9)

![Screenshot - Playlists](https://raw.githubusercontent.com/streamlist/streamlist/master/screenshot1.png)
![Screenshot - Library](https://raw.githubusercontent.com/streamlist/streamlist/master/screenshot2.png)

## Features

* **Playlists**
  * Create unlimited playlists from your library
  * Shareable link for each playlist
* **Library**
  * Filter your media by title and description
  * Add to playlists with one button
* **Import**
  * YouTube search and download
  * Built in search with queueing downloader
  * Cancel downloading/transcode jobs
* **HTTP Streaming to any device**
  * Transcodes to streamable MP4.
  * Converts any source to AAC-encoded MP4 (.m4a) audio files using ffmpeg
  * Plays in all modern web browsers, media players, and podcast apps
* **Private Podcast URLs**
  * Each Playlist has its own private podcast URL
  * Works with any podcast app on desktop or mobile
  * Enables downloading for offline access
* **Lightweight player**
  * Uses browser-native player with minimal additions
  * Keyboard shortcuts
  * `space` (play/pause) `p` (previous) `n` (next) `m` (mute) `-`/`+` (volume)
  * Remembers your volume setting
* **Simple self-hosting**
  * Public Docker image
  * Single static Go binary with assets bundled
  * Automatic TLS using Let's Encrypt
  * Redirects http to https

## Running

### 1. Get a server

**Recommended Specs**

* Type: VPS or dedicated
* Distribution: Ubuntu 16.04 (Xenial)
* Memory: 512MB
* Storage: 10GB+

**Recommended Providers**

* [OVH](https://www.ovh.com/)
* [Scaleway](https://www.scaleway.com/)

### 2. Add a DNS record

Create a DNS `A` record in your domain pointing to your server's IP address.

**Example:** `music.example.com  A  172.16.1.1`

### 3. Enable Let's Encrypt

When enabled with the `--letsencrypt` flag, streamlist runs a TLS ("SSL") https server on port 443. It also runs a standard web server on port 80 to redirect clients to the secure server.

**Requirements**

* Your server must have a publicly resolvable DNS record.
* Your server must be reachable over the internet on ports 80 and 443.

### 4. Run the static binary

Replace `amd64` with `arm64` or `armv7` depending on your architecture.

```bash

# Install ffmpeg.
$ sudo apt-get update
$ sudo apt-get install -y wget ffmpeg

# Download the streamlist binary.
$ sudo wget -O /usr/bin/streamlist https://raw.githubusercontent.com/streamlist/streamlist/master/streamlist-linux-amd64

# Make it executable.
$ sudo chmod +x /usr/bin/streamlist

# Allow it to bind to privileged ports 80 and 443.
$ sudo setcap cap_net_bind_service=+ep /usr/bin/streamlist

# Enable Let's Encrypt using your domain for automatic TLS configuration.
$ streamlist --letsencrypt --http-host music.example.com
.503869865804371e+09    info    Streamlist URL: https://music.example.com/streamlist
1.503869865804527e+09    info    Login credentials:  streamlist  /  1134423142

```

## Run the Docker Image

### 1. Install Docker

```bash
# Update apt
$ sudo apt-get update

# Remove old docker install.
$ sudo apt-get remove docker docker-engine docker.io

# Ensure we have basics for apt-get.
$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

# Add Docker's public key.
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# Add Docker's apt repo
$ sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"

# Update apt
$ sudo apt-get update

# Install Docker
$ sudo apt-get install docker-ce

# Run the hello-world test image
$ sudo docker run hello-world

```

### 2. Run the Docker image

The official image is `streamlist/streamlist`, which should run in any up-to-date Docker environment.

```bash

# Your download directory should be bind-mounted as `/data` inside the container using the `--volume` flag.
$ mkdir /home/<username>/Music

$ sudo docker create                            \
    --name streamlist --init --restart always   \
    --publish 80:80 --publish 443:443           \
    --volume /home/<username>/Music:/data   \
    streamlist/streamlist:latest --letsencrypt --http-host music.example.com

$ sudo docker start streamlist

$ sudo docker logs -f streamlist
1.503869865804371e+09    info    Streamlist URL: https://music.example.com/streamlist
1.503869865804527e+09    info    Login credentials:  streamlist  /  1134423142

```

### 3. Updating the container image

Pull the latest image, remove the container, and re-create the container as explained above.

```bash
# Pull the latest image
$ sudo docker pull streamlist/streamlist

# Stop the container
$ sudo docker stop streamlist

# Remove the container (data is stored on the mounted volume)
$ sudo docker rm streamlist

# Re-create and start the container
$ sudo docker create ... (see above)

```

