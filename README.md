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

## Images

## [How to install the project](docs/install.md)

## [How to create the project](docs/create.md)

## How to use phpqa

## How to use behat

## How to use the mailer server
