---
layout: page
title: "Models"
nav_order: 1
parent: "Definitions"
---

# Models
{: .no_toc }

## Table of Contents
{:.no_toc .text-delta} 

1. TOC
{:toc}

## Fields
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
parameter that is required to define a field. Other optional parameters are:

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

## Relations
Relations are also defined under `properties,` but instead of the field type, you have to specify related entity and 
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
Jinn supports many-to-one, one-to-many, and many-to-many relations.

### One-to-many and Many-to-one
Jinn will automatically generate a DB field to hold the relation in the proper table.
For a many-to-one relation, as with `author`, the field `author_id` will be added to Posts table.
For a one-to-many relation, as with `comments`, the field `post_id` will be added to Comments table, 
even though the relation is not defined on the comments entity.

Of course, any relation of this type is two-sided, but you decide which model will have the property.
You may also define "opposite" relations in both entities so that both models will have relation accessors.
In the example above, we could add the following property to the Comment:
```yaml
post: { entity: Post, relation: many-to-one }
```
    
Keep in mind that such opposite relations will match each other only when the property on "many" side of the relation
is named after the related entity, i.e., Post.comments references Comment.

Let's now update the User model to have access to all user's posts:
```yaml
User:
  properties:
    ...
    posts: { entity: Post, relation: one-to-many }    
Post:
  properties:
    ...
    author: { entity: User, relation: many-to-one }
```
With a definition like this, Jinn has no way to tell whether these two properties define the same relation
or two different relations, so it will stay on a safe side, and you'll end up having two relation fields in your Posts
table: `author_id` and `user_id`. In order to resolve this issue, you should advise Jinn which non-standard property name 
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
Without disambiguation, both relations will try to add "user_id" field to the Posts DB table, which will cause an error.
So, one or both of them should define `via`, regardless of the fact whether you have opposite relations or not:
```yaml
User:
  properties:
    ...
    authoredPosts: { entity: Post, relation: one-to-many, via: author }    
    approvedPosts: { entity: Post, relation: one-to-many, via: approver }    
```
For consistency, `via` parameter may also be defined on a many-to-one relation, which will alter the DB field name, 
but there are not many real usage scenarios for that.

### Many-to-many
Many-to-many relations are defined in a similar way:
```yaml
User:
  properties:
    ...
    watchedPosts: { entity: Post, relation: many-to-many }
```

As usual, Jinn will handle the rest automatically and will generate a pivot table and corresponding properties.
This relation may also be defined on both entities if needed:
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

A pivot table name is generated using alphabetically sorted names of entities (same as in Eloquent relations), so in this case, it will be `PostUser`.
You can give a more meaningful name to it using the already known parameter `via`, but keep in mind to do this on both sides if you have them.
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
`via` parameter also allows you to create several many-to-many relations between the same entities: just give each relation its own via table name.

Sometimes it is handy to store some additional information inside the pivot. In order to achieve this, we need to create 
an entity for the pivot and then set its name in`via` parameter. 
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

## Extends, Implements, and Traits
Due to the way Jinn generates classes, it is not possible to directly change the base class of a model.
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

---
Next: [Views]({% link definitions/views.md %})
