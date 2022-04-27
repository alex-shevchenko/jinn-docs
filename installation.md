---
title: "Installation"
layout: page
nav_order: 2
---

# Installation (Laravel)
## Setup via composer

```shell 
composer require jinn/laravel
```

## Publish Jinn config file

```shell 
php artisan vendor:publish --provider="Jinn\Laravel\JinnServiceProvider"
```
## Create Jinn folder structure
Default structure:  
```
jinn 
- def
- gen 
```
An alternative structure can be configured via `config/jinn.php`.
Further, this guide will use a default configuration. Changing the Jinn configuration 
should result in corresponding changes to the next steps.

## Configure autoload
Edit `composer.json`, locate autoload section, and add a line as follows:
```json
"autoload": {
    "psr-4": {
        ...
        "JinnGenerated\\": "jinn/gen/" 
    }
}
```
Then ask the composer to update autoload files:

```shell
composer dump-autoload
```

---

Next: [Getting Started]({% link getting-started.md %})
