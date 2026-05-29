IP: 134.199.158.115
Domain: dash.jacobat.me
Explainer video: video.jacobat.me

Prerequisites:
* Purchased Domain, with nameserver control
* Ubuntu VM LTS 24.04
* Cloudflare Account
# Overview
We'll be running the following applications
* **Tabbycat Debate**, for managing debating competitions
* **MUN Controller** (custom), for managing Model United Nations competitions
* **Portainer**, for managing docker containers via UI rather than CLI
* **NGINX Proxy Manager**, for SSL and DNS
# Install Docker
Docker is the containerization platform we'll be using for setting up our web applications.

Run
`sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

Verify Docker is running
`sudo systemctl status docker`

(Optional) Verify the installation
`sudo docker run hello-world`

# Run Portainer
Portainer makes it easy to manage Docker containers through a graphical UI. This is crucial in an environment where non-technical volunteers may have to restart containers urgently.

The Portainer docker container can be run through cli, however docker compose provides a level of granularity that I prefer to docker run. 

Make a directory for Portainer's docker compose file
`mkdir portainer`

Enter that directory
`cd portainer`

Create a docker compose file
`touch docker-compose.yaml`

Edit that file
`nano docker-compose.yaml`
```

services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce:lts
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    ports:
      - 9443:9443
      - 8000:8000  # Remove if you do not intend to use Edge Agents

volumes:
  portainer_data:
    name: portainer_data

networks:
  default:
    name: portainer_network

```

Alternatively, download the compose file from Portainer directly using curl
`curl -L https://downloads.portainer.io/ce-sts/portainer-compose.yaml -o portainer-compose.yaml`

Check to see if Portainer is running on its default port. Replace with server's public IPV4
`http://172.0.0.1:9443`

Go through Portainer's account creation flow to create your admin account. 

# Setting up Tabbycat
Tabbycat is a powerful open-source debate tabulation software, [used in a legitimate capacity by myself at WADL](https://draw.wadl.org). For this, we will be deploying a docker container, create account credentials, and load a dummy competition.

Like before, let's create a directory for Tabbycat and enter that directory
```
mkdir tabbycatdebate
cd tabbycatdebate
```
Tabbycat is a smaller project, so there is no docker image on the Docker Hub. This means we have to download its source code and then run a docker container.

Latest:
`git clone https://github.com/TabbycatDebate/tabbycat/

Stable:
`wget https://github.com/TabbycatDebate/tabbycat/archive/refs/tags/v2.11.1.tar.gz`

If you download the stable version, you need to extract the contents.
```
tar -xvzf v2.11.1.tar.gz
```

Now we've downloaded the projects source code, we can deploy it to docker. The source code comes with a premade docker-compose file.

```
docker compose up -d --build
```

> [!NOTE] Mount path error
> Sometimes, the container will fail to build because the stable release uses an old Postgres schema. Fix this by removing the "/data" path from the postgres bind mount in the Docker Compose file 

Check tabbycat is running. Note that you might have to change the application's port, as it collides with the port used by Portainer. Because Portainer is a larger project, it's generally best practice it gets port priority. I chose a random port for Tabbycat.

`http://172.0.0.1:8042`

Follow the account creation flow, then select a dummy competition. I chose 88 team BP.

Tabbycat is now set up and working!

# Setting up NGINX Proxy Manager
==Ensure you have access to change your DNS records==

NGINX is what we'll use for our