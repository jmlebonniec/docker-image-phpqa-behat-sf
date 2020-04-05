# How to create the project

## First install docker and docker-compose
For docker: follow the install instruction here: https://docs.docker.com/install/linux/docker-ce/ubuntu/
For docker-compose: follow the install instruction here: https://docs.docker.com/compose/install/

## Create docker-compose config file
We install docker with a front, a mailer, a mariadb and a selenium testing container.

Create a docker-compose.yml file with (don't forget to replace {{ your_container_name }} with your name, for exemple "sfdocker"):

```
version: '3.7'
services:
    traefik:
        image: traefik:1.7
        container_name: {{ your_container_name }}
        command: --docker --docker.exposedbydefault=false
        ports:
            - "80:80"
        volumes:
            - /var/run/docker.sock:/var/run/docker.sock
            - ./docker/traefik/traefik.toml:/traefik.toml:ro
        networks:
            - {{ your_container_name }}

    {{ your_container_name }}-front:
        image: thecodingmachine/php:7.3-v2-apache-node8
        container_name: {{ your_container_name }}-front
        labels:
        - traefik.enable=true
        - traefik.backend=front
        - traefik.frontend.rule=Host:${HOST_{{ your_container_name }}}
        - traefik.port=80
        volumes:
        - ./:/var/www/html:cached
        environment:
#            STARTUP_COMMAND_1: yarn encore dev --watch &
            APACHE_DOCUMENT_ROOT: ./public
            PHP_EXTENSION_XSL: 1
            PHP_INI_MEMORY_LIMIT: 1g
        networks:
            - {{ your_container_name }}

    {{ your_container_name }}-mariadb:
        image: mariadb
        container_name: {{ your_container_name }}-db
        ports:
            - "3306:3306"
        volumes:
        - ../dump:/var/dump
        - mysql_data:/var/lib/mysql
        environment:
            MYSQL_ALLOW_EMPTY_PASSWORD: "yes"
            MYSQL_DATABASE: customer_area
        command:
            'mysqld --innodb-flush-method=fsync'
        networks:
            - {{ your_container_name }}

    {{ your_container_name }}-mailer:
        image: mailhog/mailhog
        container_name: {{ your_container_name }}-mailer
        labels:
            - traefik.enable=true
            - traefik.backend=mailer
            - traefik.frontend.rule=Host:${HOST_MAILER}
            - traefik.port=8025
        networks:
            - {{ your_container_name }}

    {{ your_container_name }}-selenium-testing:
        image: selenium/standalone-chrome:3.11
        container_name: {{ your_container_name }}-selenium-testing
        volumes:
        - /dev/shm:/dev/shm
        networks:
            - {{ your_container_name }}

volumes:
    mysql_data:
        driver: local

networks:
    {{ your_container_name }}:
        name: {{ your_container_name }}
        driver: bridge
```


## Create .env config file
Create a .env file with (don't forget to replace {{ your_container_name }} with your name):

```
# In all environments, the following files are loaded if they exist,
# the later taking precedence over the former:
#
#  * .env                contains default values for the environment variables needed by the app
#  * .env.local          uncommitted file with local overrides
#  * .env.$APP_ENV       committed environment-specific defaults
#  * .env.$APP_ENV.local uncommitted environment-specific overrides
#
# Real environment variables win over .env files.
#
# DO NOT DEFINE PRODUCTION SECRETS IN THIS FILE NOR IN ANY OTHER COMMITTED FILES.
#
# Run "composer dump-env prod" to compile .env files for production use (requires symfony/flex >=1.2).
# https://symfony.com/doc/current/best_practices/configuration.html#infrastructure-related-configuration

###> docker-compose ###
HOST_SFDOCKER=sfdocker.local
HOST_MAILER=mailer.sfdocker.local
###< docker-compose ###
```

## Launch docker containers

Run command bellow:
```
docker-compose up -d
```

If you make the right things, all the container are running with "done" status. Otherwise check the docker-compose.yml file

## Install the framework

Access to the front container
```
docker exec -ti {{ your_container_name }}-front bash
```

In the first please connect to the application ec-front and apply this :
```
npm install
yarn install
```

Inside the front container
```
composer create-project symfony/website-skeleton project "4.3.99"
```

Why symfony 4.3? because we want to use phpqa (and other tools like that), and, for now, it's only working in this version.

Move symfony project in the root:
```
mv project/* .
rm -rf project
```

Inside the front container
```
composer require phpunit/phpunit --dev
composer require sensiolabs/security-checker
composer require edgedesign/phpqa --dev
composer require friendsofphp/php-cs-fixer --dev
composer require phpstan/phpstan --dev
composer require phpstan/phpstan-beberlei-assert --dev
composer require phpstan/phpstan-doctrine --dev
composer require phpstan/phpstan-phpunit --dev
composer require phpstan/phpstan-symfony --dev
composer require behat/behat --dev
composer require behat/mink --dev
composer require behat/mink-browserkit-driver --dev
composer require behat/mink-extension --dev
composer require behat/mink-goutte-driver --dev
composer require behat/mink-selenium2-driver --dev
composer require behatch/contexts --dev
composer require doctrine/doctrine-fixtures-bundle --dev
composer require friends-of-behat/symfony-extension --dev
composer require nette/neon --dev
composer require doctrine/doctrine-migrations-bundle
composer require symfony/webpack-encore-bundle
```

Add the file "phpstan.neon" in the root 
```
parameters:
    symfony:
        container_xml_path: '%currentWorkingDirectory%/var/cache/dev/srcApp_KernelDevDebugContainer.xml'
    excludes_analyse:
        - %rootDir%/../../../src/Migrations/*
includes:
    - %currentWorkingDirectory%/vendor/phpstan/phpstan-beberlei-assert/extension.neon
    - %currentWorkingDirectory%/vendor/phpstan/phpstan-symfony/extension.neon
    - %currentWorkingDirectory%/vendor/phpstan/phpstan-doctrine/extension.neon
    - %currentWorkingDirectory%/vendor/phpstan/phpstan-phpunit/extension.neon
```

Add the file ".phpqa.yml" in the root
```
phpqa:
    analyzedDirs: ./src,./tests
    buildDir: build/
    ignoredDirs: vendor
    ignoredFiles: ""
    report: true
    output: file
    execution: parallel
    tools: phpmetrics:0,phploc:0,phpcs:0,php-cs-fixer:0,phpmd:0,pdepend:0,phpcpd:0,security-checker:0,phpstan:0
    verbose: false
    extensions:
        - php

phpcs:
    standard: PSR2
    # can be overriden by CLI: phpqa --tools phpcs:1
    allowedErrorsCount: 0
    # number of allowed errors is compared with warnings+errors, or just errors from checkstyle.xml
    ignoreWarnings: true
    lineLimit: 300
    # https://github.com/squizlabs/PHP_CodeSniffer/wiki/Reporting
    reports:
        cli:
        - full
        file:
            # checkstyle is always included and overriden
            checkstyle: checkstyle.xml
            # you can include custom reports (https://github.com/wikidi/codesniffer/blob/master/reports/wikidi/Summary.php#L39)
            # ./vendor/owner/package/src/MySummaryReport.php: phpcs-summary.html

php-cs-fixer:
    allowedErrorsCount: null
    # http://cs.sensiolabs.org/#usage
    rules: '@Symfony'
    allowRiskyRules: false
    # by default the tool is runned in dry-run mode (no fixers are applied)
    isDryRun: true
    # alternatively you can define path to your .phpcs_file (rules/allowRiskyRules config is ignored)
    config: .php_cs.dist

phpmd:
    allowedErrorsCount: 0
    standard: phpmd.xml

phpstan:
    allowedErrorsCount: null
    level: max
    standard: phpstan.neon
```

Add runnable scripts to composer.json for phpqa and suites
```
    "scripts": {
    	...
        "cs-check": "phpcs",
        "cs-fix": "phpcbf",
        "phpstan": "phpstan analyse src tests -c phpstan.neon --level=max --no-progress -vvv",
        "phpqa": "phpqa --verbose",
        "phpqa-hardcore": "phpqa --tools=phpmetrics:0,phploc:0,phpcs:0,php-cs-fixer:0,phpmd:0,pdepend:0,phpcpd:0,psalm:0,security-checker:0,phpstan:0",
        "php-cs-fixer": "php-cs-fixer fix src/ tests/ --rules=@Symfony --dry-run --verbose --config=.php_cs.dist",
        "php-cs-fixer-fix": "php-cs-fixer fix src/ tests/ --verbose --config \"/var/www/html/.php_cs.dist\"",
        "phpqatest": "phpqa --analyzedDirs src tests --tools phpcs --report",
        "behat": "php bin/console app:import-translation --truncate && behat",
        "fix-ci": [
            "@cs-fix",
            "@php-cs-fixer-fix",
            "@phpqa"
        ],
        ...
```

Add in your /etc/hosts (host machin):
```
127.0.0.1 sfdocker.local
127.0.0.1 mailer.sfdocker.local
```