
# Installation (Laravel)
## Setup via composer
    composer require jinn/jinn-laravel@dev-master
## Publish Jinn config file
    php artisan vendor:publish --provider="Jinn\Laravel\JinnServiceProvider"
## Create Jinn folder structure
Default structure:  
```
jinn 
- def
- gen 
```
Alternative structure can be configured via `config/jinn.php`.
Further guide will use default configuration. Changing Jinn configuration 
should result in corresponding changes to the next steps.

## Configure autoload
Edit `composer.json`, locate autoload section and add a line as follows:
```json
"autoload": {
    "psr-4": {
        ...
        "JinnGenerated\\": "jinn/gen/" 
    }
}
```
Then ask composer to update autoload files:

```shell script
composer dump-autoload
```

# Getting Started
## Basic Definition
Copy the following definition into `jinn/def/entities.yaml` or create your own.
```yaml
User:
  class:
    extends: Illuminate\Foundation\Auth\User
  properties:
    name: string
    email: { type: string, unique: true }
    password: string
    avatar_filename: { type: string, required: false }

Post:
  properties:
    content: text
    author: { entity: User, relation: many-to-one }
    published_at: datetime
    comments: { entity: Comment, relation: one-to-many }

Comment:
  properties:
    content: text
    author: { entity: User, relation: many-to-one }
    published_at: datetime
``` 
## Generation (Laravel)
Ask Jinn to generate the files
```shell script
php artisan jinn
```

Inspect generated files under `app` and `jinn/gen` folders.

# Full Guide
## Definitions
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

## Model
### Fields
Model properties are defined under `properties` key of the Entity. For example, the User entity below has three properties defined:
```yaml
User:
  properties:
    name: string
    email: { type: string, unique: true }
    password: string
```
Simple form `name: string` will generate a required (non-nullable) string field.
As you can see, this is a shorthand form for `name: { type: string }`. `type` is the only
parameter which is  required to define a field. Other optional parameters are:

|Property   |Default        |Description                                                                        |
|--         |--             |--                                                                                 |
|required   |true           |Affects validation and makes DB field non-nullable                                 |
|default    |null           |Sets the default DB field value                                                    |
|length     |depends on type|Only valid for types which are mapped to `VARCHAR`, sets field's length in the DB  |
|unique     |false          |Creates unique DB index and affects generated validation                           |
|index      |false          |Creates non-unique DB index                                                        |

Supported types are:

|Jinn type  |Mapped DB type |Comment                                |   
|--         |--             |--                                     |
|string     |varchar        |Default length: 255                    |
|email      |varchar        |Adds validation, default length: 100   |
|int        |int            |                                       |
|bigint     |unsigned bigint|                                       |
|float      |float          |                                       |
|bool       |tinyint        |                                       |
|date       |date           |                                       | 
|datetime   |datetime       |                                       |

### Relations
Relations are also defined under `properties` but instead of the field type you have to specify related entity and 
relation type as shown below:
```yaml
Post:
  properties:
    author: { entity: User, relation: many-to-one }
    comments: { entity: Comment, relation: one-to-many }
    content: text

Comment:
  properties:
    content: text
    author: { entity: User, relation: many-to-one }
```
Jinn supports many-to-one, one-to-many and many-to-many relations.

#### One-to-many and Many-to-one
Jinn will automatically generate DB field to hold the relation in the proper table.
For many-to-one relation, as with `author`, the field `author_id` will be added to Posts table.
For one-to-many relation, as with `comments`, the field `post_id` will be added to Comments table, 
even thou the relation is not defined on the comments entity.

Of course, any relation of this type is two-sided, but you decide which model will have the property.
You may also define "opposite" relations in both entities, so that both models will have relation accessors.
In the example above we could add the following property to the Comment:
```yaml
post: { entity: Post, relation: many-to-one }
```
    
Keep in mind that such opposite relations will match each other only when the property on "many" side of the relation
is named after the related entity, i.e. Post.comments references Comment.

Let's now update User model to have access to all user's posts:
```yaml
User:
  properties:
    ...
    posts: { entity: Post, relation: one-to-many }    
Post:
  propertis:
    ...
    author: { entity: User, relation: many-to-one }
```
With such definition Jinn has no way to tell, whether these two properties define a same relation
or two different relations, so it will go on a safe side and you'll end up having two relation fields in your Posts
table: `author_id` and `user_id`. In order to resolve this issue you should advice Jinn which non-standard property name 
is used for the many-to-one relation. So we should update the definition as follows:
```yaml
User:
  properties:
    ...
    posts: { entity: Post, relation: one-to-many, via: author }    
Post:
  propertis:
    ...
    author: { entity: User, relation: many-to-one }
```

Another scenario is when you need to have several one-to-many relations to the same entity, for example:
```yaml
User:
  properties:
    ...
    authoredPosts: { entity: Post, relation: one-to-many }    
    approvedPosts: { entity: Post, relation: one-to-many }    
```
Without disambiguation both relations will try to add "user_id" field to Posts DB table which will cause an error.
So, one or both of them should define `via`, regardless of the fact whether you have opposite relations or not:
```yaml
User:
  properties:
    ...
    authoredPosts: { entity: Post, relation: one-to-many, via: author }    
    approvedPosts: { entity: Post, relation: one-to-many, via: approver }    
```
For consistency, `via` parameter may also be defined on many-to-one relation, which will alter the DB field name, 
but there are not much real usage scenarios for that.

#### Many-to-many
Many-to-many relations are defined in a similar way:
```yaml
User:
  properties:
    ...
    watchedPosts: { entity: Post, relation: many-to-many }
```

As usual, Jinn will handle the rest automatically and will generate a pivot table and corresponding properties.
This relation may also be defined on both entities, if needed:
```yaml
User:
  properties:
    ...
    watchedPosts: { entity: Post, relation: many-to-many }
Post:
  properties:
    ...
    watchers: { entity: User, relation: many-to-many }
```

Pivot table name is generated using alphabetically sorted names of entities (same as in Eloquent relations), so in this case it will be `PostUser`.
You can give a more meaningful name to it using already known parameter `via`, but keep in mind to do this on both sides, if you have them.
```yaml
User:
  properties:
    ...
    watchedPosts: { entity: Post, relation: many-to-many, via: Watchers }
Post:
  properties:
    ...
    watchers: { entity: User, relation: many-to-many, via: Watchers }
```
`via` paramter also allows to create several many-to-many relations between same entities: just give each relation it's own via table name.

Sometimes it is handy to store some additional information inside the pivot. In order to achieve this we need to create 
an entity for the pivot and then set it's name in `via` paramter. 
```yaml
User:
  properties:
    ...
    watchedPosts: { entity: Post, relation: many-to-many, via: Watchers }
Post:
  properties:
    ...
    watchers: { entity: User, relation: many-to-many, via: Watchers }
Watchers:
  properties:
    watched_at: datetime
```
Note that you only need to define the additional data you want to save within the pivot. The
relation fields will be added automatically.

### Extends, implements and traits
Due to the way Jinn generates classes it is not possible to directly change the base class of a model.
Also, while you still can define interface implementations and use traits, you may also want Jinn to do this for you.
All of this can be done by using `class` parameters group on an entity as follows:
```yaml
User:
  class:
    extends: Illuminate\Foundation\Auth\User
    traits:
      - Laravel\Sanctum\HasApiTokens
      - Illuminate\Notifications\Notifiable
    implements:
      - Illuminate\Contracts\Auth\CanResetPassword
      - Illuminate\Contracts\Auth\MustVerifyEmail
  ...
```

## Views
Before diving into API generation we need to describe Views concept. 
Generally, a View is a subset of model's fields. Most often Views are used 
to limit the data which is returned by certain API methods. Jinn also uses views 
to define a subset of fields accepted by `create` and `update` API methods.
There are no restriction for using views outside of Jinn-generated API, 
which is why they have their own definition section.

Let's check an example:
```yaml
User:
  properties:
    name: string
    email: { type: string, unique: true }
    password: string
  views:
    default:
      - name
      - email
```
Here we define a simple view named `default` which can be used to return user information excluding the `password`.

Larger entities may require having several similar views. Copying almost all fields into each view will be 
against Jinn's ideology, so there is a better way to achieve this: extend views.
```yaml
User:
  properties:
    name: string
    email: { type: string, unique: true }
    password: string
  views:
    create: ~ #copy all fields into the view
    update: 
      extends: create
      remove:
        - email #we don't want email to be editable
    get:
      extends: update
      remove:
        - password
      add:
        - email
```
Each view defined here is generated as a class and may be used in the API definition, 
as described below, or elsewhere.

## API
### Basics
Let's just go ahead and enable API for Posts:
```yaml
Posts:
  ...
  api: ~
```
That's it. This definition will tell Jinn to generate standard set of API methods for Post entity:

|Method type|HTTP Action|Default route  |
|--         |--         |--             |
|list       |GET        |/posts         |
|get        |GET        |/posts/{id}    |
|create     |POST       |/posts         |
|update     |PUT        |/posts/{id}    |
|delete     |DELETE     |/posts/{id}    |

Before proceeding, let's enable routing. 

#### Enable Routing (Laravel)
As soon as you add APIs to your definitions, the next generation
will create API routing file under `jinn/gen/routes/api.php`.
The file must be required from the main `routes/api.php` as follows
```php
$jinn = require(base_path() . '/jinn/gen/routes/api.php');
Route::group($jinn);
```
You may also add any middleware to the group as follows
```php
Route::middleware(['first', 'second'])->group($jinn);
```

### Adjust the default list of methods
In simple scenarios you would want to stick with default methods configuration, but might need to limit the list of generated methods.
One way is to disable some of the methods:
```yaml
Posts:
  ...
  api: 
    delete: false #generate all methods exception delete
```
Or you can set the list of enabled methods:
```yaml
Posts:
  ...
  api: #generate only list and get methods
    list: ~
    get: ~
```
However, in more complex projects you would want to configure each API method
in detail. So let's dive into API customization.

### API method customization
#### Method name
By default API method name matches it's type, but you may want to change the name. 
In this case you specify method name as the key and must add the type attribute:
    
```yaml
status: { type: get }
```

It will affect the method name in controller and also the route, which will change to:

    /posts/<method-name>        #for list and create methods
    /posts/{id}/<method-name>   #for delete, get and update methods
    
One of usage scenarios is to create several methods of the same type, but with different configuration.

#### Method route
Also, you may change the method route completely using route attribute:

```yaml
get: { route: /get-all-posts }
```

Route string must be in a format which will be accepted by your framework's routing system.

#### Authentication and Authorization
Some methods require the user to be authenticated, which can be ensured using `auth` attribute:
```yaml
get: { auth: true } 
```

And if authorization is required, use `policy` to define it. There are two automatic policies available at the moment:
* `owner` policy checks if the user is an owner of the entity (not applicable for `list` and `create` methods). 
Validation is performed by comparing a given entity property with the current user.
* `role` policy checks if the user is of a defined role.

Both policies can be used together, in such case user should either be an owner or has a defined role.

> **Note (Laravel)**. As Laravel does not have a built-in roles system, Jinn expects user to have `role` property
> and checks it

Example:
```yaml
Post: 
  properties:
    ...
    author: { entity: User, relation: many-to-one }
  api:
    get: 
      auth: true 
      policy:
        owner: author
        role: admin
```

If built-in policies are not suitable, just define an empty policy:
```yaml
get: { policy: ~ }
```
An empty policy method and authorization code will generated and then you can define your policy logic by overriding it.
   
#### Fields
By default, GET API methods return all fields of the entity and POST/PUT methods accept all fields of the entity.
In order to restrict some fields from being returned/updated you can use `fields` or `view` parameters. 

Using the `fields` you define the list for the method directly:
```yaml
User: 
  ...
  api:
    get:
      fields:
        - name
        - email
```

And using the `view` you can reference an earlier defined [View](#views):
```yaml
User:
  ...
  views:
    get:
      - name
      - email
  api:
    get: { view: get }
    list: { view: get }
```

Note, that in order to include related entities in the response you'll have to extend the generated code.

### Relations
Jinn currently does not have full support of defining a relationship based controller. Instead Jinn allows you to
generate an API method, which will return the list of related objects for an entity. See example below:
```yaml
Post:
  properties:
    ...
    comments: { entity: Comment, relation: one-to-many }
  api:
    comments:
      type: relatedList
      relation: comments
      view: get
``` 

As you can see in this example a special method type `relatedList` is used.
`relation` parameter sets relation name and can be omitted if it matches the method name.
All the other parameters are same as for the usual `list` method. As you would expect, `view` parameter references target entity views.

## Generated code (Laravel)
This section of documentation is going to describe generated classes, their public and protected interfaces.
It is yet to be written. Until then fill free to inspect the generated code as it is quite simple and self-explanatory.

## Advanced topics
### Command Line Options (Laravel)
As mentioned earlier in this guide, Jinn is executed using the following command:

```shell script
php artisan jinn
```
    
Normally, Jinn will at first make sure that there are no pending migrations, then generate everything needed
and then execute any migrations it just generated. In some (rare) cases you might want Jinn to just do it's generation job, but 
do not run any migrations. It can be achieve using `--dont-migrate` option:

```shell script
php artisan jinn --no-auto-migrate
```
    
With this option pending migrations checks and migrations execution will be skipped, but Jinn will still generate it's migrations.
Alternatively you might use `--no-migrations` option:

```shell script
php artisan jinn --no-migrations
```
    
In this case Jinn will skip any database related tasks: migration checks, generation and execution.

### Repository
One approach to repository management is to exclude `jinn/gen` from your commits using .gitignore or similar. 
Then you'll need to execute Jinn as part of your build/deploy process. 
It would be a good idea to run Jinn with `no-migrations` option during builds.

Another approach is to commit `jinn/gen` contents into repository. In this case you should move Jinn
package to the `require-dev` section of your `composer.json` to exclude it's code from your builds.

### Multiple definition files
As the amount of entities and APIs in your project grows, your definitions might become quite big. 
To help manage such growing definitions Jinn provides you with a possibility to split definitions into multiple files. 
Jinn reads all files found under it's definitions folder and combines them before generation is started. 
This way entities defined in one file may freely reference entities defined in the other files.

There are two levels on which Jinn can combine the definitions:

* Entities level: you can place different entities into different files
* Entity sections level: i.e. you can define properties in one file, API in the other file and so on

This implies following possible strategies to split your definitions:

* Split entities into several files by their functional domains
* Or even place every entity into a separate file
* Alternatively you may place all your model definitions into `models.yaml` file and all API definitions into `api.yaml`
* You may place `classes` and `views` sections to one of those files or have separate files for them
* Or you may further combine the approaches as your project grows

Note, that you cannot split the definitions on a lower level. I.e. you cannot define part of entity fields 
in one file and other fields in the other file.

# TODO
- [X] Models:
    - [ ] More field types?
    - [ ] Different types for `id`
- [X] APIs:
    - [ ] Related controller
    - [ ] Additional methods: associated/disassociate, other?
    - [ ] Specs
- [X] Migrations:
    - [ ] MongoDB
- [ ] Admin panel
- [ ] File watcher
- [ ] Symfony implementation
