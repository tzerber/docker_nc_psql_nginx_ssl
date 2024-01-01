# Examples of adding services to the proxy: 
- [Examples of adding services to the proxy:](#examples-of-adding-services-to-the-proxy)
  - [Service running on the same host as the proxy container:](#service-running-on-the-same-host-as-the-proxy-container)
  - [Service running on different host in the local network of the proxy container:](#service-running-on-different-host-in-the-local-network-of-the-proxy-container)
    - [Service running inside docker on different host behind the NAT:](#service-running-inside-docker-on-different-host-behind-the-nat)
    - [Service running on other host in the NAT outside of docker :](#service-running-on-other-host-in-the-nat-outside-of-docker-)


## Service running on the same host as the proxy container:
Define the service as shown in the example compose file. The most imporant thing is the `container_name` and the `networks:` section. 
The networks section should always include the correct external name for the proxy's network, i.e. :
```yaml
networks:
  proxy:
    name: proxy-manager_proxy_network
    external: true
```

Then open the proxy web management interface and add the new service to the proxy :

Go to the "Hosts" menu, and from there go to "Proxy hosts"
![Nginx login](/media/nginx-3.png?raw=true "Host")

Setup a new host and make sure to enter the correct domain name, container name and port :
> [!TIP]
> The example uses nginx, and nginx exposes port 80 by default. If the service is using other port, for example 9000, enter that port in the proxy configuration.

![Nginx login](/media/nginx-6.png?raw=true "NewHost")

Then go on the SSL tab and set up Lets Encrypt as shown on the image.

![Nginx login](/media/nginx-5.png?raw=true "SSL")


> [!IMPORTANT]  
> Use a valid email address for LetsEncrypt. LE will send an email if there's a problem with the certificate in the future.

Click on save and wait for it to finish. That's it, your new service is added and it should be reachable.

## Service running on different host in the local network of the proxy container:

There are two general options for this case, service running inside docker on another host, or service running on another host without docker. In both scenarios the service must be reachable on port that is used only by that service, and nothing else. Needless to say, this port must not be reachable over the internet(i.e. no NAT should exist from the firewall to that ip/host : port ).

---

### Service running inside docker on different host behind the NAT:
Define the service as shown in the example compose file. The most imporant thing is the `ports` section. 

```yaml
    ports:
      - 4321:80
```
In the example the service (nginx as example) will be reachable over port 4321 on the host. 

The proxy configuration is the same as the others, with the difference that for "Forward hostname" instead of the container name we enter the IP/Hostname of the host. (Screenshots below)

> [!TIP]
> Since this is a relatively basic example, i suggest keeping it to hard-coded IPs, for more complex scenarios where DNS / DHCP / Assigned hostnames are involved instead of hardcoded IPs a hostname can be used.
> 
---

### Service running on other host in the NAT outside of docker :

If for example another service (like Windows IIS web-app or a RaspberryPI with some service like home-asssistant) needs to be proxied to the internet the configurations is absolutely the same as the example with docker above. All it's needed is the correct IP/Hostname and the port of the service.

---

Go to the "Hosts" menu, and from there go to "Proxy hosts"
![Nginx login](/media/nginx-3.png?raw=true "Host")

Setup a new host and make sure to enter the correct domain name, IP/Hostname and port :

![Nginx login](/media/nginx-7.png?raw=true "NewHost")

Then go on the SSL tab and set up Lets Encrypt as shown on the image.

![Nginx login](/media/nginx-5.png?raw=true "SSL")
