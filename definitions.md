---
layout: page
title: "Definitions"
nav_order: 4
has_children: true
---

# Definitions
Jinn's main concept is Entity. Entity has a name and a definition, which describes
it's model, API, admin panel, and some more parameters which are described later.

Definitions are read from configured definitions folder. It is expected that all files in that folder are valid YAML files.

For small projects it is reasonable to use a single file for all definitions. 
You can name it `entities.yaml`, for example. 

For larger projects there are several strategies available for using multiple files.
They are covered [later in this documentation](#multiple-definition-files).

## Database and Migrations
As mentioned, Jinn manages it's database tables. It means that whenever the model is changed,
Jinn generates and executes corresponding migrations. As a consequence, Jinn cannot run safely if there 
are pending migrations in the project. So, generation will be aborted if pending migrations are detected.
This behavior can be [overridden](#command-line-options-laravel), however, it is not recommended.

Since the migrations are automatic, in order to avoid accidental data loss, Jinn never generates `drop` commands.
If the field is removed from the model, Jinn will just make sure that the field is marked as nullable, so that the 
application can continue inserting data to the table. If the model is removed Jinn will do nothing.
You may create migrations to drop columns or tables manually.

All database names are generated automatically and cannot be changed. Following Laravel's Eloquent, Jinn adds
`id` primary key field to all models. At this stage generated `id` field is always an auto-increment integer.
Support for other types is yet to be implemented.

