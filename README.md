
# Configuring Mail Containers on Raspberry Pi OS

## Launch and configure mail services
In its current state, this repository provides a minimally functional mail environment based on separate postfix (SMTP), dovecot (IMAP), and rainloop (webmail) containers.

1. Install git (if required) with `sudo apt install git` and then clone this repo locally.
2. Navigate to the sub-directories for each service and build a tagged image. 
    - In `docker-mail/postfix` run `docker build -t postfix .` 
    - In `docker-mail/dovecot` run `docker build -t dovecot .` 
    - In `docker-mail/rainloop` run `docker build -t rainloop .` 
3. Create a user-defined container network: 
    - `docker network create --driver bridge --opt com.docker.network.bridge.name=docker-mail docker-mail`
4. Launch the containers as shown below:
    - `docker run -itd --name dovecot --network docker-mail -p 143:143 dovecot`
    - `docker run -itd --name postfix --network docker-mail -p 25:25 postfix`
    - `docker run -itd --name rainloop --network docker-mail -p 80:80 rainloop`
5. In a browser, navigate to `http://<hostname>/?admin` to configure a new domain:
    - Configure a new domain called `test.pi`
        - Default admin: `admin` / `12345`
        - Test domain: `test.pi`
        - IMAP: `dovecot.docker-mail:143`
        - SMTP: `postfix.docker-mail:587`
    - Set the default domain under the Login tab to `test.pi`
6. Navigate to `http://<hostname>/` to access webmail.
    - Test user: `pi@test.pi` / `password`

## Restarting your containers

If you don't complete this guide in one sitting, you may encounter a surprise on rebooting your Pi. By default, your containers will not restart when the docker service restarts. At the same time, you won't be able to issue the `docker run` command from _Step 4_ again because the containers already exist. To examine this, run `docker ps` to see a list of running containers. Next, run `docker ps -a` to see a list of all containers including those that have exited.

One option is to delete the containers using the `docker rm` command, though by doing this, you'll lose any updates you made to the container and will need to restart at _Step 4_.

Fortunately, docker provides us with an easy method to bring our existing containers back online by running:

`docker start postfix dovecot rainloop`

The base command here is `docker start` and the arguments we are passing are the names we assigned to each container when we created them with `docker run`.

> **Note:** While you're at it, update your container to restart automatically at boot as shown in [**Launch Containers at Boot**](#launch-containers-at-boot).

## Customize your mail servers

### Change the domain name and hostname 

The postfix container can be customized by passing in a set of _environment variables_ at runtime with the `-e` argument to `docker run`. To make this change, we will tear down the postfix container and start a new version with a custom domain:

> Rather than serving mail for test.pi, this example will be configured for gradebook.pi. 

1. Stop the existing container with `docker stop postfix`. This command will return an error response reading `No such container` if the container is already stopped.
2. Remove the existing container by running `docker rm postfix`. Once again, postfix refers to the name we assigned in our original `docker run` commmand.
3. Relaunch the new container with the `mail_domain` and `smtp_hostname` environment variables by running `docker run -d --name postfix -e "mail_domain=gradebook.pi" -e "smtp_hostname=mail.gradebook.pi" --network docker-mail -p 25:25 postfix`

### Add / remove users
You can add or remove users by editing the included `users_template` and copying it into the running container at `/etc/dovecot/users`.

- `docker cp LOCAL_FILE CONTAINER_NAME:/etc/dovecot/users`

> CONTAINER__NAME = dovecot if you have followed examples above*

The general format of the user file is one `username:password` pair per line. The username should not include the _@domain_ suffix. The template demonstrates the format of adding a user with a plaintext password. You can leverage the dovecot container image to create a stronger password hash:

- `docker run -it --rm IMAGE_NAME doveadm pw -s PBKDF2`

> IMAGE_NAME = dovecot if you have followed examples above*

## Basic Container Management

### Restart suspended containers
At the moment, restarting the device will result in the containers being suspended. You can restart a suspended container by calling `docker start` with the container name. 

### Delete existing containers
If at any point you need to rebuild the containers, remove the current version of the containers using `docker rm -f dovecot postfix rainloop`

### Launch containers at boot
By default, containers will remain stopped if we reboot or otherwise restart the docker service. We can override this default by adding the `--restart always` parameter to a `docker run` command. 

> **Note**: A common mistake with docker beginners is to try to add a parameter at the end of a `docker run` command. This will fail since the `docker run` command expects the last argument to be the name of the image.

If you already have a running container, there's no need to remove it in order to change the default behavior. Instead, use `docker update` to set the restart behavior for your current container(s), e.g.:

`docker update --restart always postfix rainloop dovecot`
