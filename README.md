# General purpose PHP images for Docker
This repository contains a set of developer-friendly, general purpose PHP images for Docker.

You can enable or disable the extensions using environment variables.
You can also modify the php.ini settings using environment variables.
2 types available: slim (no extensions preloaded) or fat (most common PHP extensions are built-in)
3 variants available:## [How to install the project](docs/install.md) CLI, apache and fpm
Fat images are bundled with Supercronic which is a Cron compatible task runner. Cron jobs can be configured using environment variables
Fat images come with Composer and Prestissimo installed
All variants can be installed with or without NodeJS (if you need to build your static assets).
Everything is done to limit file permission issues that often arise when using Docker. The image is actively tested on Linux, Windows and MacOS

Each image come with:
- phpqa library: for using coding standard, 
- behat (goutte and selenium): for the no regression of your code,
- mailer server: a server where your email would be catch in development environment

# Images

| Name        | PHP           | Symfony  |
| ------------- |:-------------:| -----:|
| [jmlebonniec/docker-image-phpqa-behat-sf]| 7.3 | 4.3 |


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

# How to use phpqa

phpqa (PHP Quality Assurance) PHP quality checking tools, to allow you to code to a particular standard and easily spot errors and violations.

After your development launch the command cli:

```
composer phpqa
```

# How to use behat

For testing the no-regression of your development, use the cli command:

```
behat
```

# How to use the mailer server

Go to the url: http://mailer.{your-name-project}.local/ and see the mailer interface, every mail send from your site find here
