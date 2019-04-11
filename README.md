# Nginx Docker Letsencrypt Deployment Template

This repository provides some sample resources for deploying a Nginx web server alongside Letsencrypt Certbot that automates the process of SSL certificate renewals.

The bulk of these configurations are adapted from an excellent Medium article found [here](https://medium.com/@pentacent/nginx-and-lets-encrypt-with-docker-in-less-than-5-minutes-b4b8a60d3a71).

Readers are encouraged to read this document for a more thorough understanding of how the configuration works. This document anticipates some familiarity with Docker and Nginx and will focus on modifying the configurations for new deployments.

## What's In The Box

`docker-compose.yml` - A sample Docker Compose configuration file that contains the Nginx and Certbot configurations. Feel free to add other services to this configuration file.

`data` - Configurations for Nginx and Certbot that ultimately end up getting bind mounted to the appropriate containers.

`init-letsencrypt.sh` - A shell script for retrieving the first instance of the SSL certificate.

## Configuration

### Setting Up Docker Service URLs

The main Nginx configuration file located at `data/nginx-conf/nginx.conf` does not require very much configuration. For any containerized services that will be run as a part of the configuration simply add the container name and associated port as an `upstream` directive. The example below shows two services (service-1 and service-2) that expose port 8000. Note that the names are simply the hostnames for each container within the Docker network.

```vim
upstream service-1 {
    server service-1:8000;
}
upstream service-2 {
    server service-2:8000;
}
```

### Setting Up App Configuration

Note that in the last line of `data/nginx-conf/nginx.conf` there is a directive to include other configuration files located in `/etc/nginx/conf.d`. Inside the container, the directory `data/nginx` will be bound to that location. By default, there is a file named `app.conf` located in this directory. This configuration will be the most heavily modified.

Replace all instances of `myserver.com` with the Fully Qualified Domain Name of the server (Lines 3, 20, 23, 24, and 47).

For each service, add the following directives (replacing the location and proxy pass name with the appropriate values). With this configuration, each of the associated locations will be accessible as `base-url/service-name`. For example, `service-1` would be accessed as `https://myserver.com/service-1`. Note that the default configuration will redirect all HTTP requests to HTTPS.

```vim
location /service-1 {
    rewrite ^/service-1(.*) /$1 break;
    proxy_pass http://service-1;
    proxy_set_header    Host                $http_host;
    proxy_set_header    X-Real-IP           $remote_addr;
    proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
}


location /service-2 {
    rewrite ^/service-2(.*) /$1 break;
    proxy_pass http://service-2;
    proxy_set_header    Host                $http_host;
    proxy_set_header    X-Real-IP           $remote_addr;
    proxy_set_header    X-Forwarded-For     $proxy_add_x_forwarded_for;
}
```

### Getting Initial SSL Certificate

The final step in the installation process requires modifying and running the `init-letsencrypt.sh` script in order to get the first SSL certificate and setup the necessary certbot components. Open the first script in your favorite text editor and add the FQDNs that certificates should be generated for. Next, provide the email address of the individual who is registering for the SSL certificate. The `data_path` variable should not be modified if no changes were made to this script.

Once these configuration steps have been completed, the containers should run as expected and SSL certificates will be automatically renewed without user intervation.