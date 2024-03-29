# Installing Nextcloud MariaDB and Portainer-ce dockers in Linux VM

Using Debian 11 64-bit  
using network install from minimal cd amd64 version from https://www.debian.org/CD/netinst/  
install without desktop options  
install with standard system utilities  
root password - secret  
user - texas  
password - secret  
set static IP

# install sudo
ref https://milq.github.io/enable-sudo-user-account-debian/
Install sudo with apt install sudo.
```
apt install sudo
```
Add the user account to the group sudo with /sbin/adduser username sudo. Where username is your user account.
```
/sbin/adduser texas sudo
```

# install docker
ref https://docs.docker.com/engine/install/debian/#install-using-the-repository  
running from console as root  

```
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/debian \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin

sudo service docker start
systemctl status docker
sudo docker run hello-world
```

# install portainer-ce docker 
ref https://docs.portainer.io/start/install/server/docker/linux

```
sudo docker volume create portainer_data
sudo docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

check that portainer has started  
```
docker ps
```

1st time login to portainer create admin user  
username: admin  
password: secret

# method 1 - install containers and set up database connection
this uses the standard nextcloud docker, which does not include SMB/CIFS external folders

install mariadb docker from portainer  
ref https://hub.docker.com/_/mariadb  
- add new volume "mariadb"
- select MariaDB from App Templates
- set root password - secret
- click on + Show advanced options
- map host 3306 to container 3306
- map container folder /var/lib/mysql to volume mariadb
- deploy
- optional for people with OCD - stop container and rename to mariadb

nextcloud docker install from portainer  
ref https://github.com/nextcloud/docker  
- add volume "nextcloud"
- create container
- name nextcloud
- image - nextcloud:latest
- publish network port
- map host 80 to container 80
- map container folder /var/www/html to volume nextcloud
- hit deploy

initial setup
- open nextcloud webpage
- create admin user
- username: admin
- password: secret
- storage & database option - select mysql/mariadb
- database user - root
- database password - secret (as set during mariadb docker install)
- database name - nextcloud
- database host - nextcloud docker ip & port ie 172.17.0.3:3306
- hit install
- install recommended apps

# method 2 - docker-compose / stacks in portainer to build docker image with SMB support
docker-compose ref - https://github.com/nextcloud/docker#base-version---apache  
adding features ref - https://github.com/nextcloud/docker#adding-features  

using the full nextcloud dockerfile from https://github.com/nextcloud/docker/tree/master/.examples/dockerfiles for adding all optional packages (except libreoffice)  
for this to work, the Dockerfile location needs to be specified in docker-compose.yml by replacing `image: nextcloud:latest` with `build: /data/nextcloud_full`  

create nextcloud_data folder with the required dockerfiles for portainer  
we run these commands from the host  

```
cd /var/lib/docker/volumes/portainer_data/_data
mkdir nextcloud_full
cd nextcloud_full
wget https://raw.githubusercontent.com/limawaken/nextcloud-mariadb-portainer-docker/master/Dockerfile
wget https://raw.githubusercontent.com/limawaken/nextcloud-mariadb-portainer-docker/master/supervisord.conf
```

the official example files
```
wget https://raw.githubusercontent.com/nextcloud/docker/master/.examples/dockerfiles/full/apache/Dockerfile
wget https://raw.githubusercontent.com/nextcloud/docker/master/.examples/dockerfiles/full/apache/supervisord.conf
```


go back to portainer and add stack  
paste docker-compose.yml and deploy

# additional nextcloud setup

# add External Storages and LDAP / AD integration
configure AD integration  
server: 192.168.8.6  
port: 389  
admin username: adm1n.cloud@SMSB.local  
password: secret  
Base DN: dc=SMSB,dc=local  
object classes: person  
from groups: SMSB Cloud Access  
login attributes: AD username  
groups: empty  
under Expert settings - internal username attribute: sAMAccountname  


# additional notes

# updating nextcloud docker

ONLY FOR METHOD 1  
ref https://github.com/nextcloud/docker#update-to-a-newer-version  
pull new image  
```
docker pull nextcloud:latest
```
remove old container
```
docker stop <your_nextcloud_container>
docker rm <your_nextcloud_container>
```
recreate docker container
```
docker run -d -p 8080:80 -v nextcloud:/var/www/html nextcloud
```

in portainer just remove the container and create it again.

for method 2 (docker-compose custom build)  
in portainer go to stacks  
edit nextcloud stack  
update stack, pull and re-deploy  

# updating portainer
ref https://docs.portainer.io/start/upgrade/docker
```
docker stop portainer
docker rm portainer
docker pull portainer/portainer-ce:latest
docker run -d -p 8000:8000 -p 9443:9443 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

# install vmware tools
ref https://docs.vmware.com/en/VMware-Tools/12.2.0/com.vmware.vsphere.vmwaretools.doc/GUID-C48E1F14-240D-4DD1-8D4C-25B6EBE4BB0F.html
```
sudo apt-get update
sudo apt-get install open-vm-tools
```