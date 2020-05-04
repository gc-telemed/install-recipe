# Installation steps 
## For dicom4chee stack, nginx, keycloak and OHIF viewer. 

- Install docker and git:

```
sudo apt install docker docker-compose git -y
```

Clone this recipe repo:

```
git clone https://github.com/gc-telemed/install-recipe.git
```

## Setup
1. Create the docker folder if it doesn't exist: `sudo mkdir -p /opt/docker`

2. Copy app.json and site.conf to /opt/docker: `sudo cp app.json site.conf /opt/docker`

3. Edit /etc/hosts (e.g. `sudo nano /etc/hosts`) Add the IP you get from `ifconfig en0 inet` to your hosts file and alias it to the name 'dockerhost'. 

You may need to replace `en0` with your network interface name e.g. 

```
$ ifconfig 
enp1s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.122.14  netmask 255.255.255.0  broadcast 192.168.122.255
        inet6 fe80::9df6:4dea:ce28:663d  prefixlen 64  scopeid 0x20<link>
        ether 52:54:00:b8:97:b3  txqueuelen 1000  (Ethernet)
        RX packets 63572  bytes 169724384 (169.7 MB)
        RX errors 0  dropped 14  overruns 0  frame 0
        TX packets 40494  bytes 4586282 (4.5 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 1442  bytes 149512 (149.5 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1442  bytes 149512 (149.5 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

In this case one would choose inet IP 192.168.122.14 from `enp1s0`. This is how your `/etc/hosts` file could look after the edit:

```
127.0.0.1       localhost
127.0.1.1       risav                          
192.168.122.14  dockerhost

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

3. Check if docker service/daemon is running:

```
sudo service docker status
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: https://docs.docker.com
```

If it shows inactive like the above output, start the service (`sudo service docker start`)

Finally, run `docker-compose up` in the folder where you cloned this repo. In some cases you may get the following error:

```
ERROR: Couldn't connect to Docker daemon at http+docker://localunixsocket - is it running?

If it's at a non-standard location, specify the URL with the DOCKER_HOST environment variable.
```

In this case, you will need to add your user to the docker group and log in with your user again as shown below. 

```
sudo usermod -aG docker ${USER}
su - $USER
```

You can now switch back to the folder where you cloned this repo and run `docker-compose up` again.


4. [Add the OpenID Connect Client for dcm4chee](https://github.com/dcm4che/dcm4chee-arc-light/wiki/Run-all-archive-services-with-secured-UI-on-a-single-host#register-the-archive-ui-as-oidc-client-in-keycloak)

5. Add an OpenID Connect Client for OHIF. Follow the same instructions as for dcm4chee, but use the root URL of just http://dockerhost/

## Testing
* Log in to the OHIF Viewer at http://dockerhost using the 'Login with Keycloak' button.
* Log in to the dcm4chee Archive at http://dockerhost/dcm4chee-arc/ui2
* Upload or push (e.g. STOW-RS or C-MOVE) some studies into dcm4chee. These will become visible and loadable in the OHIF Viewer.
* Load Keycloak's administration console at http://dockerhost/auth/admin/dcm4che/console. Now you can create Users and assign them Roles.
* Logging into one service will give you access to the other services.
* Try using curl http://dockerhost/dcm4chee-arc/aets/DCM4CHEE/rs/studies
