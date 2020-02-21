# docker-nginx-proxy

This project combines [nginx-proxy](https://github.com/jwilder/nginx-proxy) with [docker-letsencrypt-nginx-proxy-companion](https://github.com/JrCs/docker-letsencrypt-nginx-proxy-companion) to manage hosting one or more sites from a single server.
This configuration handles all public-facing HTTP/HTTPS traffic as well as HTTPS certificate management via [Let's Encrypt](https://letsencrypt.org/).

## Setting everything up

- Clone this repository into a new directory on your server, e.g., `/var/www/nginx-proxy/`.
- Copy `.env.example` to `.env` and update its settings.
- (Optional) Copy the `*.conf.example` files in `/data/conf.d/` or add your own config files as necessary. Any files in this directory that don't end in `*.conf` will be ignored.

## HTTPS

This configuration will automatically retrieve and update certificates for sites defined with a `LETSENCRYPT_HOST` environment variable as shown below.

You can also manually manage third-party certificates by adding them to `/data/certs/`.
Create certificate and private key files for each domain in this directory with names like `example.com.crt` and `example.com.key`.

See the [nginx-proxy documentation](https://github.com/jwilder/nginx-proxy#ssl-support) for additional details.

## Adding virtual hosts

The `nginx-proxy` container in this project is the only container that should have ports 80 and 443 exposed to the public.
There's no need to expose these ports on application-specific containers since `nginx-proxy` will automatically forward requests to the appropriate application container via the internal Docker network.

Ultimately you might end up with multiple layers of nginx (from `nginx-proxy` to an app-specific instance of nginx), but this is the preferred configuration because it makes it easier to contain configuration settings for each project.

Here's an example of what an app's Docker compose file might look like.
This would be located at `/var/www/your-app/docker-compose.yml`.

```yml
version: '3.7'

services:

  nginx:
    image: nginx

    environment:
      VIRTUAL_HOST: example.com,www.example.com
      LETSENCRYPT_HOST: example.com,www.example.com

    # Connect this container to the shared nginx-proxy network as well as the default network.
    networks:
      - default
      - nginx-proxy

  # Add your app's other services (e.g., PHP, MySQL, etc.) here.

networks:
  nginx-proxy:
    external:
      name: nginx-proxy
```
