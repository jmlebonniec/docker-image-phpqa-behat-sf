# How to install the project

## First install docker and docker-compose
For docker: follow the install instruction here: https://docs.docker.com/install/linux/docker-ce/ubuntu/
For docker-compose: follow the install instruction here: https://docs.docker.com/compose/install/

## After install source project
```
composer create-project jmlebonniec/docker-image-phpqa-behat-sf your-project-name
```

Add in your host the host:
```
127.0.0.1 {your-name-project}.local
127.0.0.1 mailer.{your-name-project}.local
127.0.0.1 mariadb.{your-name-project}.local
```
In the project folder, copy .env.template to .env

Be attentive to replace correctly {your-name-project} with your project name!!!

Replace sfdocker terms with your own reference (project name for example) in the docker-compose.yml

```
composer install
```

You can start docker with command
```
docker-compose up -d
```
