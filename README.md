# myStrom Plug Prometheus Exporter

Cloned and updated for myStrom from https://github.com/geerlingguy/shelly-plug-prometheus

I have a [myStrom Plug](https://mystrom.com/wifi-switch/). I don't want to flash a different firmware on it just to get a Prometheus endpoint. Though I might do it someday if I'm not lazy.

I want a Prometheus endpoint where I can scrape metrics for the myStrom Plug.

So JG built this.

Could I use MQTT and use some sort of preconfigured broker / metrics generator? Yeah probably. But it's literally easier (if you don't already use MQTT and know how to set it up) to just write a little metrics scraper like this for a one-off.

## How to Use

This thing is a PHP script that runs in a Docker container. That's it.

You could even run the thing without using Docker, it doesn't care, it's just a PHP script.

### Set environment variables

I recommend adding a basic HTTP auth username and password to your myStrom Plug to give at least minimal security.

Run the PHP script inside Docker like so:

```
docker run -d -p 9924:80 --name myStrom-plug \
  -e MYSTROM_HOSTNAME='my-myStrom-plug-host-or-ip' \
  -e MYSTROM_HTTP_USERNAME='username' \
  -e MYSTROM_HTTP_PASSWORD='password' \
  -v "$PWD":/var/www/html \
  php:8-apache
```

Or you can set it up inside a docker-compose file like so:

```yaml
---
version: "3"

services:
  myStrom-plug:
    container_name: myStrom-plug
    image: php:8-apache
    ports:
      - "9924:80"
    environment:
      MYSTROM_HOSTNAME='my-myStrom-plug-host-or-ip'
      MYSTROM_HTTP_USERNAME='username'
      MYSTROM_HTTP_PASSWORD='password'
    volumes:
      - './:/var/www/html'
    restart: unless-stopped
```

If you run the script outside of Docker, make sure you set the necessary environment variables.

After it's running, test that it's working with `curl`:

```
$ curl http://localhost:9924/metrics
# HELP power Current real AC power being drawn, in Watts
# TYPE power gauge
power 149.88
# HELP temperature Current being drawn, in Celsius
# TYPE temperature gauge
temperature 22.62
# HELP The average of energy consumed per second from last call this request
# TYPE average gauge
average 150.36
```

If you get an error, check your environment variables and make sure they're correct, and make sure the Docker container (and the host it's on) can access your myStrom Plug over HTTP!

## License

MIT.

## Author

[cloned and updated for myStrom](https://www.jeffgeerling.com).
