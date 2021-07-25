# Private Docker Registry

## Setup

Before starting, you will need a domain. In `nginx/registry.conf`, replace the code below with it:
```
...

server {
  ...
  server_name myregistrydomain.com;
  
...
```

To manage users in your private registry, you need to run the commands below:
```
# If your machine doesn't recognize the htpasswd command, run:
sudo apt-get -y install apache2-utils

# If the nginx folder doesn't have the registry.password file, you must run:
htpasswd -Bc registry.password USERNAME

# Otherwise, you can run:
htpasswd registry.password USERNAME
```

Whenever a user has their credentials created or updated, it's necessary to recreate the containers:
```
docker-compose stop
docker-compose rm
docker-compose up -d
```

## Enabling ssh

If you keep ssl disabled in `nginx/registry.conf`, all users will have to [configure Docker to allow connections to your 
insecure docker registry](https://docs.docker.com/registry/insecure/#deploy-a-plain-http-registry). To avoid this, in addition
to the secutiry isues that can happen, generate certificates to enable ssh on your private registry. In case you choose
to use self-signed certificates, you will need to configure a few more things in addition to what is described below. You can find
more info [here](https://docs.docker.com/registry/insecure/#use-self-signed-certificates).

First, put you certificates in the `nginx` folder and then uncomment the lines below in `nginx/registry.conf`:
```
...

server {
  ...
  # ssl on;
  # ssl_certificate /etc/nginx/conf.d/domain.crt;
  # ssl_certificate_key /etc/nginx/conf.d/domain.key;
...
```

You can rename your certificates files to match those described above, or you can change the path in the `conf` file to
match the names of the certificates files.

## Running

To confirm that Nginx is properly forwarding traffic to your registry container, navigate to your domain in a browse window
and access `https://your_domain:5043/v2`, if you see an empty JSON object, `{}`, so everything is set up configured. Now you
can run `docker login your_domain:5043` and push all your private images. Don't forget to tag your images correctly, adding the
`your_domain:5043/*` prefix.

## More info

If you got stuck in any of the steps above, you can find more info below:
- https://docs.docker.com/registry/recipes/nginx/
- https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-20-04
