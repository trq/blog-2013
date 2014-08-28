---
layout: post
title: Composer and local git repos
categories:
    - blog
---
Just had a question in #laravel on freenode which reminded me, I get a bit sick
of explaining this. Using local repositories, or any kind of repository with
composer is simple. All you need do is tell composer where the package's
repsitory lives.


    "repositories": [
        {
            "type":"vcs",
            "url":"/Users/trq/src/somerepo"
        }
    ]

Now, as long as that repository had a valid composer.json file with a name
defined, compooser can use it. As an example, my 'somerepo' repository has a
composer.json containing:


    {
        "name": "trq/somerepo",
        "autoload": {
            "psr-4": {
                "SomeRepo\\": ["src/"]
            }
        },
        "extra": {
            "branch-alias": {
                "dev-develop": "1.0.x-dev"
            }
        }
    }

So, now, all I need do is add that to my dependencies per usual.

    "require": {
        "php": ">=5.3.3",
        "symfony/symfony": "2.5.*",
        "doctrine/orm": "~2.2,>=2.2.3",
        "doctrine/doctrine-bundle": "~1.2",
        "twig/extensions": "~1.0",
        "symfony/assetic-bundle": "~2.3",
        "symfony/swiftmailer-bundle": "~2.3",
        "symfony/monolog-bundle": "~2.4",
        "sensio/distribution-bundle": "~3.0",
        "sensio/framework-extra-bundle": "~3.0",
        "incenteev/composer-parameter-handler": "~2.0",
        "trq/somerepo":"~1.0.@dev"
    }

Then install:

    composer update trq/somerepo

Done.
