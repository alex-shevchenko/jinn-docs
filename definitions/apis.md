---
layout: page
title: "APIs"
nav_order: 3
parent: "Definitions"
---

# APIs
{: .no_toc }

## Table of Contents
{: .no_toc .text-delta }

1. TOC
{: toc }


## Basics
Let's just go ahead and enable API for Posts:
```yaml
Posts:
  ...
  api: ~
```
That's it. This definition will tell Jinn to generate a standard set of API methods for the Post entity:

|Method type|HTTP Action|Default route  |
|--         |--         |--             |
|list       |GET        |/posts         |
|get        |GET        |/posts/{id}    |
|create     |POST       |/posts         |
|update     |PUT        |/posts/{id}    |
|delete     |DELETE     |/posts/{id}    |

Before proceeding, let's enable routing. 

### Enable Routing (Laravel)
As soon as you add APIs to your definitions, the next generation
will create an API routing file under `jinn/gen/routes/api.php`.
The file must be required from the main `routes/api.php` as follows
```php
$jinn = require(base_path() . '/jinn/gen/routes/api.php');
Route::group($jinn);
```
You may also add any middleware to the group as follows
```php
Route::middleware(['first', 'second'])->group($jinn);
```

## Adjust the default list of methods
In simple scenarios, you would want to stick with default methods configuration but might need to limit the list of generated methods.
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
However, in more complex projects, you would want to configure each API method
in detail. So let's dive into API customization.

## API method customization
### Method name
By default, the API method name matches its type, but you may want to change the name. 
In this case, you specify method name as the key and must add the type attribute:
    
```yaml
status: { type: get }
```

It will affect the method name in controller and also the route, which will change to:

    /posts/<method-name>        #for list and create methods
    /posts/{id}/<method-name>   #for delete, get and update methods
    
One of the usage scenarios is to create several methods of the same type but with a different configuration.

### Method route
Also, you may change the method route completely using the route attribute:

```yaml
get: { route: /get-all-posts }
```

Route string must be in a format that will be accepted by your framework's routing system.

### Authentication and Authorization
Some methods require the user to be authenticated, which can be ensured using the `auth` attribute:
```yaml
get: { auth: true } 
```

And if authorization is required, use `policy` to define it. There are two automatic policies available at the moment:
* `owner` policy checks if the user is an owner of the entity (not applicable for `list` and `create` methods). 
Validation is performed by comparing a given entity property with the current user.
* `role` policy checks if the user is of a defined role.

Both policies can be used together. In such cases, the user should either be an owner or have a defined role.

> **Note (Laravel)**. As Laravel does not have a built-in roles system, Jinn expects the user to have `role` property
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
An empty policy method and authorization code will be generated, and then you can define your policy logic by overriding it.
   
### Fields
By default, GET API methods return all fields of the entity, and POST/PUT methods accept all fields of the entity.
In order to restrict some fields from being returned/updated, you can use `fields` or `view` parameters. 

Using the `fields`, you define the list for the method directly:
```yaml
User: 
  ...
  api:
    get:
      fields:
        - name
        - email
```

And using the `view`, you can reference an earlier defined [View](#views):
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

Note that in order to include related entities in the response, you'll have to extend the generated code.

## Relations
Jinn currently does not have full support of defining a relationship-based controller. Instead, Jinn allows you to
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

As you can see in this example, a special method type `relatedList` is used.
`relation` parameter sets the relation name and can be omitted if it matches the method name.
All the other parameters are the same as for the usual `list` method. As you would expect, `view` parameter references target entity views.

---
Next: [Generated Code]({% link generated-code.md %})
