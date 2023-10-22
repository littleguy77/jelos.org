# :octicons-stack-16: Building JELOS

JELOS is a fairly unique distribution as it is *built to order* and only enough of the operating system and applications are built for the purpose of booting and executing emulators and ports.  Developers and others who would like to contribute to our project should read and agree to the [Contributor Code of Conduct](code-of-conduct.md) and [Contributing to JELOS](index.md) guides before submitting your first contribution.

## 1. Prep

Building JELOS requires a host with 200GB of free space for a single device, or 1TB of free space for a full world build.  

!!! note "Expect your first build to take on the order of ten hours.  You will need a stable internet connection, since hundreds of packages will be downloaded from their source."  

Download errors often produce build failures with misleading error messages.  If this happens to you, see the Troubleshooting section below.

After a clean build, all subsequent builds will go much faster (minutes) since 99% of the build work is cached.

### Docker **(Recommended)**

!!! tip "Docker is the easiest and most reliable way to build JELOS." 

You need no previous experience with Docker; you merely need to install it on your build machine.  Newcomers to the project are strongly recommended to use this approach.

We recommend using Ubuntu 22.04 for the host machine, as this is well-tested and known to work.  Other distributions and operating systems might also work for Docker builds, but are untested and unsupported.

``` bash title="Install Docker using the following commands:"
sudo apt update
sudo apt install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

docker run hello-world
```

> Docker installation reference (source): [Install using the apt repository](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository) and [Linux post-installation steps](https://docs.docker.com/engine/install/linux-postinstall/).

The final command should produce a message indicating that Docker is properly installed.  If you encounter any errors, see the reference links above.

### Manual Build

Manual builds (outside of Docker) are only recommended for developers with specific needs that cannot be met by the Docker approach.  The host configuration should match the Docker container as closely as possible, running Ubuntu 22.04 with all packages listed in the [Dockerfile](https://github.com/JustEnoughLinuxOS/distribution/blob/main/Dockerfile).

### Virtual Machines

If you prefer to use a virtual machine for your build platform; keep in mind that results vary. There have been some reports of success with VMware Workstation (Player and Pro) and WSL and some unsuccessful attempts with VirtualBox. These reports are not definitive, but something to keep in mind if you prefer to use a VM.

## 2. Clone

After preparing the build machine, clone the JELOS git repository onto it.

``` bash
cd ~
git clone https://github.com/JustEnoughLinuxOS/distribution.git
```

### Selecting the Desired Branch

Once you have cloned the repo, decide whether you want to build the main branch which is more stable, or the development branch which is unstable but hosts our newest features.

|Branch|Purpose|
|----|----|
|main|Stable JELOS sources|
|dev|Unstable JELOS sources|

To check out our development branch, cd into the project directory and checkout `dev`.

``` bash
cd distribution
git checkout dev
```

### Filesystem Structure
We have a simple filesystem structure adopted from parent distributions CoreELEC, LibreELEC, etc.

``` bash
.
├── build.JELOS-DEVICE.ARCHITECTURE
├── config
├── distributions
├── Dockerfile
├── licenses
├── Makefile
├── packages
├── post-update
├── projects
├── release
├── scripts
├── sources
└── tools
```

**build.JELOS-DEVICE.ARCHITECTURE**

Build roots for each device and that device's architecture(s).  For ARM devices JELOS builds and uses a 32bit root for several of the cores used in the 64bit distribution.

**config**

Contains functions utilized during the build process including architecture specific build flags, optimizations, and functions used throughout the build workflow.

**distributions**

Distributions contains distribution-specific build flags and parameters and splash screens.

**Dockerfile**

Used to build the Ubuntu container used to build JELOS.  The container is hosted at [https://hub.docker.com/u/justenoughlinuxos](https://hub.docker.com/u/justenoughlinuxos).

**licenses**

All of the licenses used throughout the distribution packages are hosted here.  If you're adding a package that contains a license, make sure it is available in this directory before submitting the PR.

**Makefile**

Used to build one or more JELOS images, or to build and deploy the Ubuntu container.

**packages**

All of the packages used to develop and build JELOS are hosted within the packages directory.  The package structure documentation is available [here](packages.md).

**post-update**

Anything that is necessary to be run on a device after an upgrade should be added here.  Be sure to apply a guard to test that the change needs to be executed before execution.

**projects**

Hardware-specific parameters are stored in the projects folder.  Anything that should not be included on every device during a world build should be isolated to the specific project or device here.

**release**

The output directory for all of the build images.

**scripts**

This directory contains all of the scripts used to fetch, extract, build, and release the distribution.  Review Makefile for more details.

**sources**

As the distribution is being built, package source files are fetched and hosted in this directory.  They will persist after a `make clean`.

**tools**

The tools directory contains utility scripts that can be used during the development process, including a simple tool to burn an image to a usb drive or sdcard.

## 3. Build

### Building Device Images

Building JELOS is easy.  From the root of your local repository, issue one of the `make` commands listed below, depending on the desired device and whether you are using Docker.

| Devices | Dependency | Docker Command | Manual Command |
| ---- | ---- | ---- | ---- |
|AMD64||`make docker-AMD64`|`make AMD64`|
|RK3588||`make docker-RK3588`|`make RK3588`|
|RK3326||`make docker-RK3326`|`make RK3326`|
|RK3566||`make docker-RK3566`|`make RK3566`|
|RK3566-X55|RK3566|`make docker-RK3566-X55`|`make RK3566-X55`|
|S922X||`make docker-S922X`|`make S922X`|
|ALL DEVICES||`make docker-world`|`make world`|

> Devices that list a dependency require you to build the dependency first, since that build will be used as the root of the device you are building.  You only need to build the dependency once after cloning or cleaning the dependency.

For example, the following command uses Docker to build the AMD64 image.  

``` bash
make docker-AMD64
```

### Rightsized Builds

JELOS supports various build variables which alter the behavior of the distribution for specific purposes including debugging, or hosting containers.  The options are defined below and are passed on the make command line.  Ex. `BASE_ONLY=true make docker-RK3566`.

|Build Option|Default|Function|
|----|----|----|
|EMULATION_DEVICE|yes|Builds EmulationStation and all emulators if `yes`. Builds EmulationStation and NO emulators if `no`.|
|ENABLE_32BIT|yes|Builds a 32bit root and includes it in the image.  Needed for 32bit cores and applications.|
|BASE_ONLY|false<sup>1</sup>|Builds only the bare minimum packages.  Includes Weston on supported devices.  Does not build EmulationStation.|
|CONTAINER_SUPPORT|no|Builds support for running containers on JELOS.|

> Note: <sup>1</sup> this property will change to yes/no for consistency in a future release.

### Env Variables

For development builds, you can use the following env variables to customize the image or change build time functionality. To make them globally available to the builds, add them to `${HOME}/.JELOS/options`.

|Variable|Function|
|----|----|
|LOCAL_SSH_KEYS_FILE|Enables using ssh public keys for access without the root password.|
|LOCAL_WIFI_SSID|The SSID of the network the device should connect to automatically.|
|LOCAL_WIFI_KEY|The WIFI authentication key for the connection."|
|SCREENSCRAPER_DEV_LOGIN|Login information for screenscraper.fr.|
|GAMESDB_APIKEY|Login information for thegamesdb.net.|
|CHEEVOS_DEV_LOGIN|Login information for retroachievements.org.|
|CLEAN_PACKAGES|Allows specifying packages to clean during a build.|

#### SSH Keys

``` bash
export LOCAL_SSH_KEYS_FILE=~/.ssh/jelos/authorized_keys
```

#### WiFi SSID and password

``` bash
export LOCAL_WIFI_SSID=MYWIFI
export LOCAL_WIFI_KEY=secret
```

#### Screenscraper, GamesDB, and RetroAchievements

To enable Screenscraper, GamesDB, and RetroAchievements, register at each site and apply the api keys in `${HOME}/.JELOS/options`. Unsetting one of the variables will disable it in EmulationStation. This configuration is picked up by EmulationStation during the build.

``` bash
# Apply for a Screenscraper API Key here: https://www.screenscraper.fr/forumsujets.php?frub=12&numpage=0
export SCREENSCRAPER_DEV_LOGIN="devid=DEVID&devpassword=DEVPASSWORD"
# Apply for a GamesDB API Key here: https://forums.thegamesdb.net/viewforum.php?f=10
export GAMESDB_APIKEY="APIKEY"
# Find your Cheevos Web API key here: https://retroachievements.org/controlpanel.php
export CHEEVOS_DEV_LOGIN="z=RETROACHIEVEMENTSUSERNAME&y=APIKEYID"
```

### Cleaning Additional Packages

``` bash
make docker-shell
CLEAN_PACKAGES="linux ppsspp-sa" make AMD64
exit
```
The first and last lines should be omitted if building outside of Docker.

## Troubleshooting

The very first build after a fresh checkout is the hardest.  Give yourself sufficient time to generate the first build and work through any failures before attempting to modify JELOS.

- Download errors produce misleading failure messages.  Beware of chasing red herrings.  A network failure is much more likely than a bug in the makefile, given how frequently it is tested by the CI system and other devs.
- Try cleaning the failed package (see above) and building again.
- If you suspect a download failure, manually delete the relevant package(s) from the `sources` and `build.JELOS-...` directories, to force a full package re-download and re-build.
- Exhaust all options before using `make clean` since this deletes the build cache and takes hours to regenerate.
- As a very last resort, delete the entire local repository and start over.
