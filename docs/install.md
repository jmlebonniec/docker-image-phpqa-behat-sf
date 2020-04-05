# How to install the project

## First install docker and docker-compose
For docker: follow the install instruction here: https://docs.docker.com/install/linux/docker-ce/ubuntu/
For docker-compose: follow the install instruction here: https://docs.docker.com/compose/install/

## After install source project
```
git clone git@bitbucket.org:jmlebonniec/symfony-docker.git
```

Add in your host the host:
```
127.0.0.1 sfdocker.local
127.0.0.1 sfdocker-mailer.local
```

In the project folder, copy .env.template to .env

You can start docker with command
```
docker-composer up -d
```
