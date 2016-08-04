# Sentry On-Premise

Official bootstrap for running your own [Sentry](https://getsentry.com/) with [Docker](https://www.docker.com/).

## Requirements

 * Docker 1.10.0+
 * Compose 1.6.0+ _(optional)_

## Resources

 * [Documentation](https://docs.getsentry.com/on-premise/server/installation/docker/)
 * [Bug Tracker](https://github.com/getsentry/onpremise)
 * [IRC](irc://chat.freenode.net/sentry) (chat.freenode.net, #sentry)

## Follow following steps:
```
sudo apt-get update
sudo apt-get -y upgrade
sudo apt-get install -y build-essential checkinstall
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker ${USER}

sudo apt-get install -y zip
wget https://github.com/VikramTiwari/onpremise/archive/master.zip
unzip master.zip
cd onpremise-master

docker login
REPOSITORY=vikramtiwari/sentry make build push

docker run --detach --name sentry-redis redis:3.2-alpine
docker run --detach --name sentry-postgres --env POSTGRES_PASSWORD=secret --env POSTGRES_USER=sentry postgres:9.5
docker run --detach --name sentry-smtp tianon/exim4

REPOSITORY=vikramtiwari/sentry
docker run --rm ${REPOSITORY} --help
docker run --rm ${REPOSITORY} config generate-secret-key

SENTRY_SECRET_KEY=""

# docker run --detach --rm --link sentry-redis:redis --link sentry-postgres:postgres --link sentry-smtp:smtp --env SENTRY_SECRET_KEY=${SENTRY_SECRET_KEY} ${REPOSITORY}

docker run --rm -it --link sentry-redis:redis --link sentry-postgres:postgres --link sentry-smtp:smtp --env SENTRY_SECRET_KEY=${SENTRY_SECRET_KEY} ${REPOSITORY} upgrade

docker run --detach --name sentry-web-01 -p 9000:9000 --link sentry-redis:redis --link sentry-postgres:postgres --link sentry-smtp:smtp --env SENTRY_SECRET_KEY=${SENTRY_SECRET_KEY} ${REPOSITORY} run web

docker run --detach --link sentry-redis:redis --link sentry-postgres:postgres --link sentry-smtp:smtp --env SENTRY_SECRET_KEY=${SENTRY_SECRET_KEY} --name sentry-worker-01 ${REPOSITORY} run worker

docker run --detach --link sentry-redis:redis --link sentry-postgres:postgres --link sentry-smtp:smtp --env SENTRY_SECRET_KEY=${SENTRY_SECRET_KEY} --name sentry-cron ${REPOSITORY} run cron
```

Complete docs: https://docs.getsentry.com/on-premise/
