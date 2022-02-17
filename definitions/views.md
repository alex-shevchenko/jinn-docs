---
layout: page
title: "Views"
nav_order: 3
parent: "Definitions"
---

# Views
Before diving into API generation, we need to describe the Views concept. 
Generally, a View is a subset of the model's fields. Most often, Views are used 
to limit the data which is returned by certain API methods. Jinn also uses views 
to define a subset of fields accepted by `create` and `update` API methods.
There are no restrictions for using views outside of Jinn-generated API, 
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

## Extending Views

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
as described below or elsewhere.

---
Next: [APIs]({% link definitions/apis.md %})
