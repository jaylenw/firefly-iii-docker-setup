# My Firefly-iii Docker Setup

**Goal**: Have development and production deployment environments setup
with docker-compose for the [Firefly-iii](https://www.firefly-iii.org/) application. Have the development environment support automatic updates using
[Watchtower](https://containrrr.github.io/watchtower/).

I have the development and production environment running on a server. The development environment will check for updates using Watchtower every week and applies them if a newer Firefly-iii `develop` image is found. I use the version number of the development environment to confidently upgrade my production environment manually for now when I need to. Watchtower notifies me when it finds a new release. The development environment also gives my users and me opportunities to test things out in the application and not worry about affecting our setup in the production environment. I have also configured Firefly-iii
to send out emails using [Mailgun](https://www.mailgun.com/).

## Local Testing

To get started with this, it is best that you are able to get Firefly-iii running locally
first before deploying it to your deployment environment (ex. your server).

This repository has a `development` and a `production` folder with each folder corresponding
to a different setup described by the `docker-compose.yml` file in each of them. The
production folder is configured to pull the latest stable image of Firefly-iii, while the
development folder is configured to pull the latest develop image of Firefly-iii.

Please visit, https://raw.githubusercontent.com/firefly-iii/docker/master/docker-compose.yml
to see an example `docker-compose.yml` file from the developer that maintains Firefly-iii.
The `docker-compose.yml` was used as the basis found in the files in the repository here.

### Getting Started

1.) Make sure you have [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/) installed.

2.) Clone this repository.

3.) Create the necessary environment files Docker expects.

The `development` folder requires the files `.db-env`, `.firefly-env`, and `.watchtower-env`.
The `production` folder requires the files `.db-env` and `.firefly-env`.

The directory structure should look like this:

```
development
---.db-env
---.firefly-env
---.watchtower-env
---docker-compose.yml
production
---.db-env
---.firefly-env
---docker-compose.yml
```

#### .db-env

This file is configured with the following. The application will use the user, password,
and database specified.

```
POSTGRES_USER=<user>
POSTGRES_PASSWORD=<password>
POSTGRES_DB=firefly
```

#### .firefly-env

Please visit, https://raw.githubusercontent.com/firefly-iii/firefly-iii/master/.env.example
to see an example environment file from the developer that maintains Firefly-iii.
I used the example file to help set some, not all configurations for myself. You may need to
use the example from the developer, some below, and your own modifications to get the setup
you want.

Some, but not all, notable configurations:

```
APP_ENV=local # we are running locally
APP_DEBUG=true # fine to show debug information in local environment
APP_KEY=<32-random-characters> # you can use online tools to help you generate one

TZ=America/Los_Angeles # changed this to LA

APP_URL=http://localhost # accessing the app over localhost

TRUSTED_PROXIES=** # running with docker

DB_CONNECTION=pgsql
DB_HOST=<db-host> # either fireflyiiidb-dev or fireflyiiidb-prod as indicated in the `docker-compose.yml` file
DB_PORT=5432
DB_DATABASE=firefly
DB_USERNAME=<user> # same user that is set in `.db-env`
DB_PASSWORD=<password> # same password that is set in `.db-env`

MAIL_DRIVER=mailgun # configured this because I am using Mailgun for emails
MAIL_FROM=<from-email-address> # this is important, set this using your verified Mailgun domain, example money@firefly.mydomain.com
MAILGUN_DOMAIN=<verified-mailgun-domain> # this is important, set your verified Mailgun domain, example firefly.mydomain.com
MAILGUN_SECRET=<your-mailgun-api-key>
MAILGUN_ENDPOINT=api.mailgun.net # this is fine if you are in the U.S.
```

#### .watchtower-env

Below is an example of how I have configured Watchtower to send notifications with Rocket Chat. Please refer to Watchtower's
documentation [here](https://containrrr.dev/watchtower/notifications/) to see how to configure notifications of your choice.

```
WATCHTOWER_NOTIFICATIONS=shoutrrr
WATCHTOWER_NOTIFICATION_URL=rocketchat://myserver.example.com/token/channel
```

4.) Once you have the environment settings set for local `development` make the necessary changes for local `production`.

5.) Next, run both the local `development` and `production` environments by running
`docker-compose up` in each folder. Visit `localhost:8081` for dev and and `locahost:8080`
for prod.

## Deployment Environments

Once you have tested that everything is working well locally for the development and
production environments, after minor adjustments you can proceed confidently to deploy them
to a server.

### Getting Started

1.) First, modify the environment files for deployment on a server.

#### .firefly-env

Some, but not all notable changes needed for deployment.

```
APP_ENV=production # we are running in a production environment (both prod and dev deployments)
APP_DEBUG=false # we don't want debug information showing in a production environment (both prod and dev deployments)
APP_URL=<your-domain-for-the site> # example, mydomain.com
SITE_OWNER=<your-email-addres> # best to list your email
```

2.) Make sure your server has Docker and Docker Compose installed. Clone the repo and add
your environment files for the development and production deployments added to the
corresponding directories.

3.) I strongly suggest having a proxy server in front of your Firefly-iii docker environments.

Below is an example of a reverse-proxy configuration to get you started using [NGNIX](https://www.nginx.com/).
*PLEASE CONFIGURE YOUR DEPLOYMENT WITH SSL.*

```
# configuration for firefly dev environment
upstream firefly-dev-backend {
    # the firefly-iii application in firefly-iii container
    server <localhost or whatever IP>:<port>; # ip and use the port specified in the docker-compose.yml
    keepalive 64;
}

server {
    # the virtual host name of this
    listen 80;
    server_name <your-domain-or-subdomain>;

    location / {
        proxy_pass http://firefly-dev-backend$uri$is_args$args;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        client_max_body_size 64M;
    }
}

```

4.) Start the two environments by running the following command, `docker-compose up -d` in each folder. This will
allow the applications to run in the background. Also note, Docker will restart the containers unless
they were explicitly told to shutdown (it will survive server reboots).

----------------------------------------------------------------------------------------------------------
Made with ♥ in Los Angeles CA.
