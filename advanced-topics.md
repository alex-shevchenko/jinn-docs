---
layout: page
title: "Advanced Topics"
nav_order: 6
---

# Advanced topics
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{: toc }

## Command Line Options (Laravel)
As mentioned earlier in this guide, Jinn is executed using the following command:

```shell
php artisan jinn
```
    
Normally, Jinn will first make sure that there are no pending migrations, then generate everything needed, and then execute any migrations it just generated. In some (rare) cases, you might want Jinn to just do its generation job, but do not run any migrations. It can be achieved using `--dont-migrate` option:

```shell
php artisan jinn --no-auto-migrate
```
    
With this option, pending migrations checks and migrations execution will be skipped, but Jinn will still generate its migrations.
Alternatively, you might use the `--no-migrations` option:

```shell
php artisan jinn --no-migrations
```
    
In this case, Jinn will skip any database-related tasks: migration checks, generation, and execution.

## Repository
One approach to repository management is to exclude `jinn/gen` from your commits using .gitignore or similar. 
Then you'll need to execute Jinn as part of your build/deploy process. 
It would be a good idea to run Jinn with the `no-migrations` option during builds.

Another approach is to commit `jinn/gen` contents into the repository. In this case, you should move Jinn
package to the `require-dev` section of your `composer.json` to exclude its code from your builds.

## Multiple definition files
As the amount of entities and APIs in your project grows, your definitions might become quite big. 
To help manage such growing definitions, Jinn provides you with the possibility to split definitions into multiple files. 
Jinn reads all files found under its definitions folder and combines them before generation is started. 
This way, entities defined in one file may freely reference entities defined in the other files.

There are two levels on which Jinn can combine the definitions:

* Entities level: you can place different entities into different files
* Entity sections level: i.e., you can define properties in one file, API in the other file, and so on

This implies the following possible strategies to split your definitions:

* Split entities into several files by their functional domains
* Or even place every entity into a separate file
* Alternatively, you may place all your model definitions into `models.yaml` file and all API definitions into `api.yaml`
* You may place `classes` and `views` sections to one of those files or have separate files for them
* Or you may further combine the approaches as your project grows

Note that you cannot split the definitions on a lower level, i.e., you cannot define part of entity fields 
in one file and other fields in the other file.
