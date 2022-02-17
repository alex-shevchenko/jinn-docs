---
layout: page
title: "Indexes"
nav_order: 2
parent: "Definitions"
---

# Indexes

As described in [model properties]({% link definitions/models.md %}#properties), single-column DB indexes can be
easily defined when defining fields. But to define multi-column indexes we should use a
dedicated section of the definition.
Let's consider an example where we add an entity to store the data needed for authorization via social networks:
```yaml
User:
  class:
    extends: Illuminate\Foundation\Auth\User
  properties:
    name: string
    email: { type: string, unique: true }
    password: { type: string, required: false }
    
SocialAuth:
  properties:
    provider: string
    provider_id: string
    user: { entity: User, relation: many-to-one }
  indexes:
    auth:
      - provider
      - provider_id
    user_provider:
      columns:
        - user_id
        - provider
      unique: true
```

Here we define two indexes on SocialAuth entity:
* Index named `auth` helps to find auth record when a user logs in
* `user_provider` index ensure that there are not more than one auth record for a particular user and a particular auth provider

Shorthand definition can be used for non-unique indexes, as with `auth`. The full definition is needed for unique indexes.

**Note** that index definition refers to DB columns and not to model properties. 
Thus, `user_provider` index refers to a `user_id` column, rather than the `user` property

---
Next: [Views]({% link definitions/views.md %})
