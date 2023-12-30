# Nextcloud fpm with PostgreSQL and nginx, nginx-proxy-manager with LetsEncrypt example on amd64 Linux 

This repo contains custom configurations for running nextcloud fpm + postgresql + nginx and nginx-proxy-manager + LE as separate container. It was done by request of a random person over the internet, so I did not go too hard on documentation. There are also other ways to do this, but this one is the one I'm using.

> [!WARNING]  
> At time of publishing this repository there is a known "misalignment of view" between the majority of browsers + web-servers and IANA on the topic of js/mjs files that causes issues with Nextcloud itself. I will not go in much detauls, but after a while this repo will become obsolete. I will try to respond on requests, but don't expect me to maintain this.

# Requirements: 
- [ ] Valid domain name with ability to change DNS settings / subdomains.
- [ ] Docker version 24.0.7 or newer.
- [ ] Docker Compose version v2.21.0 or newer.

# Setup: 
Open a terminal as root and clone this repo in a folder where the containers will be stored. 

```bash
mkdir docker-stuff && cd docker-stuff
git clone https://github.com/tzerber/docker_nc_psql_nginx_ssl.git .
```

A good suggestion is to have the the docker host having a static IP inside the network behind the firewall/nat. Forward port 80 and 443 from the internet to the IP of the docker host. Since this is very different depending on the network setup, this documentation will not go into details there.

> [!IMPORTANT] 
>The nginx-proxy-manager container uses port 81 for the web-management interface. 
> Port 81 on the docker host should not be accessible from the internet for safety reasons.

## Environment variables:
Edit the configurations for nextcloud in the following file:

`nextcloud\db.env` : 

```conf
POSTGRES_PASSWORD=SuperSecretPassword
POSTGRES_DB=nextcloud
POSTGRES_USER=nextcloud
```
> [!IMPORTANT]  
> There are also other configurations inside the compose files, nginx.conf and mime-types. They are not the default ones, and include custom configurations from my own instances based on experience. After some time nginx.conf and it's Dockerfile will need update once Nginx updates the mjs mime-type, but that's out of scope for this documentation. Same thing applies to Nextcloud's Dockerfile, once the check for php-bz2 is fixed the Dockerfile will need edit as well. Both of these exists because of NC28.0.1 so there shouldn't be any problems for now. 
---
## Basic Proxy configuration:

In the terminal go to the proxy container and start it.
```bash 
cd proxy
docker compose up --pull always -d
```
Make sure the new (sub)domain for Nextcloud is available and reachable, it can be verified by issuing this command in the terminal: 
```bash
dig @8.8.8.8 full.fqdn.of.the.domain.com
```

and the output should be like this:

``` bash
;; ANSWER SECTION:
full.fqdn.of.the.domain.com.    3600    IN      A       <your-server's-ip>
```

Then open a browser and go to your Host's IP addres on port 81, i.e. http://< ip >:81


> [!TIP]
> Default login are admin@example.com and password changeme.

Login to the web interface and correctly setup the login details.

![Nginx login](/media/nginx-1.png?raw=true "User")

![Nginx login](/media/nginx-2.png?raw=true "Pass")

Then go to the "Hosts" menu, and from there go to "Proxy hosts"
![Nginx login](/media/nginx-3.png?raw=true "Host")

Setup a new host and make sure to enter the correct domain name, container name and port (port is 80) 
![Nginx login](/media/nginx-4.png?raw=true "NewHost")

Then go on the SSL tab and set up Lets Encrypt as shown on the image.

![Nginx login](/media/nginx-5.png?raw=true "SSL")


> [!IMPORTANT]  
> Use a valid email address for LetsEncrypt. LE will send an email if there's a problem with the certificate in the future.

Click on save and wait for it to finish. 


## Nextcloud configuration
Build and start nextcloud and it's dependencies using this command in the terminal:

```bash 
cd ../nextcloud
docker compose up --build --pull always -d
```

Give it a moment to load, then using a browser visit the freshly created domain name using https. Create an user and password for your admin account:
![Nginx login](/media/nc-1.png?raw=true "NC1")

> [!CAUTION]
> During writing of this document and testing all the steps, i found a bug in the default apps of NC28.
>
> tl;dr Don't install the default suggested apps, Richdocuments Nextcloud office is broken again. I might include OnlyOffice at some point, because I actually use that instead of the default NC office.
![Nginx login](/media/nc-3.png?raw=true "NC3")

Once everything is done, in the terminal where nextcloud containers we're started execute the following commands :

Add missing indices (this command must be executed after every upgrade)
```bash
docker compose exec -u www-data app php occ db:add-missing-indices
```
Add default phone region:
the XX means default region for phone numbers using ISO 3166-1 country codes such as DE for Germany, FR for France. 
https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/config_sample_php_parameters.html#default-phone-region

```bash
docker compose exec -u www-data app php occ config:system:set default_phone_region --value="XX"
```

Then from the browser go to the Admin settings, then Basic settings and fill out the email configuration:
![Nginx login](/media/nc-4.png?raw=true "NC4")

> [!TIP]
> Sometimes the admin panel is slow to recognize the phone code change. If that happens, execute the command again and give it a minute to refresh itself.

## OnlyOffice Document server
ToDo

That's it, enjoy :)
