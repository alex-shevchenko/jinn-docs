---
title: "Getting Started"
layout: page
nav_order: 3
---

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

```shell
php artisan jinn
```

Inspect generated files under `app` and `jinn/gen` folders.

---
Next: [Definitions]({% link definitions.md %})
