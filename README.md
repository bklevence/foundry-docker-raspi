# foundry-docker-raspi
How I selfhosted Foundry VTT with Docker on my RasPi.
Keeping this as a mess of markdown for now, will try to comeback and clean up as well as post example files...


Image Raspi w/ Raspian (I used Pi3 B v1.2)
Enable SSH, VNC (supposedly can now be done using RasPi Imager)
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


Create docker-compose.yml for FoundryVTT (see my example)...

Point namecheap to Cloudflare, point cloudflare to public IP, enable DNSSEC, create token (edit zone dns template enable all zones).
Create A record and CNAME on cloudflare using subdomain.

Run docker compose to make FoundryVTT

Create nginx using docker, portforward 80 and 443 on router

Login to nginx and create account, create proxy host, try to enable SSL through proxy host and not ssl tab(give it a min). 


