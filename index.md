---
title: "Introduction"
layout: page
nav_order: 1
---

# Introduction
{:.no_toc}

## Table of Contents
{:.no_toc .text-delta} 

1. TOC
{:toc}

## What is Jinn

Jinn is a code generator that speeds up your work by generating Models, Migrations, and APIs 
from simple YAML-based definitions. 
What makes Jinn different from other generators is that it lets you customize the generated code 
without losing the ability to update it when definitions change.

## Why Jinn
Were you ever frustrated by the number of steps you need to take to 
add a simple field to one of the models in your project? 

For example, let's consider a simple backend for a blog, which has an API and an Admin panel.
The task is to add a string property called `category` into our `Post` entity.
This property can be freely set via API when a post is created or updated and also via the Admin panel.

Solving this task in `Laravel` will require the following steps:

1. Use CLI to create an empty DB migration
1. Write migration code
1. Run migrations
1. If you want your code completion to recognize the new field, you have to edit your model
1. If your project is not a trivial one, you are using API Resources, so you have to update 
one or several resources to include the new field in the response
1. Similarly, you probably have a Request class or classes for your `create` and `update` API calls, 
so you have to update those as well
1. Lastly, you need to update a list and a form in your admin panel, which is another one or two files

7 steps in total: you edited at least 5 files and ran 2 CLI commands.

Now, let's look at `Symfony`:

1. Symfony models are more complex, but luckily you can use CLI to add a field to the model 
1. Also, Symfony is able to generate a migration for you via another CLI command 
1. The third CLI command is needed to execute the migration
1. Most likely you are using the `JsonSerializable` interface, so you would still need to go and edit the `jsonSerialize` method in your model file
1. The last step depends on the way you implemented your API, but anyway, you'll have to edit 
at least one or two files to add the new field into `update` and `create` methods
1. Same as in with Laravel, the last step is to update your admin panel

6 steps in total: a least 3 files and 3 CLI commands

What about Jinn?
1. Edit your definition file

That's it. Jinn is watching the definitions folder. It will detect the change, update your code, and create and execute the migration

> **Note**. Jinn is in the early stages of development. 
> File watcher and admin panel are not implemented yet. 
> However, Models, Migrations, and APIs generation is there.
> API specs generation will also be added soon.

## Key Concepts
Of course, there are plenty of frameworks and libraries which serve the same purposes using various approaches.
This section describes key design decisions which make Jinn different.

### Code Generation
Unlike many API and admin panel frameworks, Jinn uses code generation, as it gives the following benefits:

* Only the code that is needed is generated, i.e., if you don't need an `update` method in your 
API, it will not be generated at all.
* The generated code does not have to be very generic, which makes it much simpler.

Symfony and Laravel frameworks also use code generation. However, they apply caching approach, i.e., generate the code at runtime.
Jinn generates the code at build-time, which means that there are no first-run performance drawbacks and lets you inspect the code easily.

All the above means that:

* Jinn carries no performance drawbacks compared to the other approaches.
* It is easy to inspect the code, understand how it works, extend and customize it.

### Base Classes
Each class generated by Jinn (except for Migrations) is split into two files: 

1. An empty class under your sources folder which extends a
1. Base class, located in a separate folder which contains all the logic

The idea is borrowed from `Propel`, a well-known in the past PHP ORM. The main class is 
generated only once and never updated, so you can use it to customize generated logic freely.
At the same Jinn can update the base class logic for you whenever the definition changes.

### Database
Currently, Jinn supports SQL databases only. MongoDB support is planned.

Jinn is designed to manage its database tables and models, i.e., it is not able to 
work with existing models and database tables. It also expects that no changes will be 
made to its database tables; otherwise, they may be overwritten. At the same time,
it is possible to have Jinn-managed and non-Jinn-managed tables in the same database.

### Frameworks
The Jinn reference implementation is made for Laravel, but it is designed 
to allow implementation for any framework. Contributors are welcome. 

Further down this guide, the sections which are specific to Laravel will be marked correspondingly.

---

Next: [Installation]({% link installation.md %})
