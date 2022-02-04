# foundry-docker-raspi
Selfhosting Foundry VTT with Docker on a RasPi with Asset Data Folders on a USB Thumbdrive.

File structure of repo assumes you copy or make docker directory in /home/pi

Keeping this as a mess of markdown for now, will try to comeback and clean up as well as post example files...

Initial help from here:
https://www.youtube.com/watch?v=ib55sgDYZbc


Image Raspi w/ Raspian (I used Pi3 B v1.2)
Enable SSH, VNC (supposedly can now be done using RasPi Imager)
Change admin password, rest of instructions assume default "pi" username.

Purchase domain from Namecheap
Create Cloudflare Account

Docker/Docker-Compose https://dev.to/elalemanyo/how-to-install-docker-and-docker-compose-on-raspberry-pi-1mo:

Prerequisites

Raspberry Pi with a running Raspbian OS
SSH connection enabled

To do this you can check Raspberry Pi Setup.
1. Update and Upgrade

First of all make sure that the system runs the latest version of the software.
Run the command:

    sudo apt-get update && sudo apt-get upgrade

2. Install Docker

Now is time to install Docker! Fortunately, Docker provides a handy install script for that, just run:

    curl -sSL https://get.docker.com | sh

3. Add a Non-Root User to the Docker Group

By default, only users who have administrative privileges (root users) can run containers. If you are not logged in as the root, one option is to use the sudo prefix.
However, you could also add your non-root user to the Docker group which will allow it to execute docker commands.

The syntax for adding users to the Docker group is:

    sudo usermod -aG docker [user_name]

To add the permissions to the current user run:

    sudo usermod -aG docker ${USER}

Check it running:

    groups ${USER}

Reboot the Raspberry Pi to let the changes take effect.
4. Install Docker-Compose

Docker-Compose usually gets installed using pip3. For that, we need to have python3 and pip3 installed. If you don't have it installed, you can run the following commands:

    sudo apt-get install libffi-dev libssl-dev
    sudo apt install python3-dev
    sudo apt-get install -y python3 python3-pip

Once python3 and pip3 are installed, we can install Docker-Compose using the following command:

    sudo pip3 install docker-compose

5. Enable the Docker system service to start your containers on boot

This is a very nice and important addition. With the following command you can configure your Raspberry Pi to automatically run the Docker system service, whenever it boots up.

    sudo systemctl enable docker

With this in place, containers with a restart policy set to always or unless-stopped will be re-started automatically after a reboot.


    docker-compose version


Install Portainer GUI https://pimylifeup.com/raspberry-pi-portainer/:

Installing Portainer on to the Raspberry Pi

Now that we have prepared our Raspberry Pi, we can finally install the Portainer software.

Luckily for us, this is a very simple process as Portainer runs within a Docker container.

1. With Docker set up and configured, we can use it to install Portainer to our Raspberry Pi.

As Portainer is available as a Docker container on the official Docker hub, we can pull the latest version using the following command.

    sudo docker pull portainer/portainer-ce:latest


This command will download the docker image to your device, which will allow us to run it.

Using “:linux-arm” at the end of the pull request, we explicitly ask for it to download the ARM version of the container.

2. Once Docker finishes downloading the Portainer image to your Raspberry Pi, we can now run it.

Telling Docker to run this container requires us to pass in a few extra parameters.

In the terminal on your Pi, run the following command to start up Portainer.

    sudo docker run -d -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest

A few of the big things we do here is first define the ports we want Portainer to have access to. In our case, this will be port 9000.

We assign this docker container the name “portainer” so we can quickly identify it if we ever needed.

Additionally, we also tell the Docker manager that we want it to restart this Docker if it is ever unintentionally offline.


Create /home/pi/docker/foundry/docker-compose.yml  for FoundryVTT (see my example in repo)...

    ---
    version: "3.8"

    services:
      foundry:
        image: felddy/foundryvtt:release
        hostname: change.me.com
        init: true
        privileged: true
        restart: "unless-stopped"
        volumes:
          - type: bind
            source: /home/pi/docker/foundry/usb
            target: /data 
        environment:
          - FOUNDRY_PASSWORD=changeme
          - FOUNDRY_USERNAME=changeme
          - FOUNDRY_ADMIN_KEY=changeme
          - FOUNDRY_PROXY_PORT=443
          - FOUNDRY_PROXY_SSL=true 
          - FOUNDRY_UID=1000
          - FOUNDRY_GID=1000
        ports:
          - target: 30000
            published: 30000
            protocol: tcp



Mount usb in /home/pi/docker/foundry/
https://raspberrytips.com/mount-usb-drive-raspberry-pi/

Once complete

Edit /etc/fstab with below, it's especially critical to apply UID GID or it will NOT work. 
```
    UUID=yourusbuuid        /home/pi/docker/foundry/usb        exfat defaults,user,uid=1000,gid=1000,noatime  0 0
```
Run docker compose to make FoundryVTT, check error logs while it's starting, make sure container is healthy once running locally!

Point namecheap to Cloudflare, point cloudflare to public IP, enable DNSSEC, create token (edit zone dns template enable all zones).
Create A record and CNAME on cloudflare using subdomain. Enable full (NOT FLEX) encryption between Cloudflare and host or nginx will not work later)

Create /home/pi/docker/nginx then install using below docker-compose.yml and config.json, portforward 80 and 443 on router

d-c.yml:
```
    version: '2'
    services:
      app:
        image: 'jc21/nginx-proxy-manager:latest'
        restart: unless-stopped
        ports:
          - '80:80'
          - '81:81'
          - '443:443'
        environment:
          DB_MYSQL_HOST: "db"
          DB_MYSQL_PORT: 3306
          DB_MYSQL_USER: "changeme"
          DB_MYSQL_PASSWORD: "changeme"
          DB_MYSQL_NAME: "npm"
        volumes:
          - ./data:/data
          - ./letsencrypt:/etc/letsencrypt
      db:
        image: 'yobasystems/alpine-mariadb:10.4.17'
        restart: unless-stopped
        environment:
          MYSQL_ROOT_PASSWORD: 'changeme'
          MYSQL_DATABASE: 'npm'
          MYSQL_USER: 'changeme'
          MYSQL_PASSWORD: 'changeme'
        volumes:
          - ./data/mysql:/var/lib/mysql
```
config.json:
```  {
          "database": {
            "engine": "mysql",
            "host": "db",
            "name": "npm",
            "user": "changeme",
            "password": "changeme",
            "port": 3306
          }
        }
```

Login to nginx and create account, create proxy host for Foundry->Cloudflare, try to enable SSL through proxy host and not ssl tab(give it a min). 



