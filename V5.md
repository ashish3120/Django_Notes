# Django Models & Migrations – Database Management Guide

> Models and Migrations are the heart of database management in Django. They allow you to define your database structure in pure Python and propagate changes to the actual database systematically and safely.

---

## Table of Contents

1. [What are Models? (`models.py`)](#1-what-are-models-modelspy)
2. [Common Field Types](#2-common-field-types)
3. [The Migrations Workflow](#3-the-migrations-workflow)
4. [Anatomy of a Migration File](#4-anatomy-of-a-migration-file)
5. [Important Interview Concepts](#5-important-interview-concepts)
6. [Database Default](#6-database-default)

---

## 1. What are Models? (`models.py`)

### The Heart of the Backend

A **Model** represents the **database structure (schema)** of your application. Rather than writing raw SQL, you define your tables as Python classes — Django handles the rest.

- Each **class** maps to a single **database table**
- Each **class attribute** maps to a **database field (column)**
- Every model must **inherit** from `models.Model`

---

### Basic Model Example

```python
# models.py
from django.db import models

class Person(models.Model):
    name  = models.CharField(max_length=100)
    age   = models.IntegerField()
    email = models.EmailField()
```

The above class tells Django to create a `Person` table with three columns: `name`, `age`, and `email`. Django also automatically adds an `id` column as the primary key.

---

## 2. Common Field Types

Django provides a rich set of built-in field types to represent different kinds of data:

| Field Type        | Use Case                                         | Key Argument         |
|-------------------|--------------------------------------------------|----------------------|
| `CharField`       | Short-to-medium text (names, titles)             | `max_length` required|
| `TextField`       | Large amounts of text (descriptions, addresses)  | —                    |
| `IntegerField`    | Whole numbers (age, quantity, count)             | —                    |
| `EmailField`      | Email addresses (validates format automatically) | —                    |
| `BooleanField`    | True / False values (active status, flags)       | —                    |
| `DateTimeField`   | Date and time combined (timestamps)              | —                    |
| `ImageField`      | Image file uploads                               | `upload_to` path     |
| `FileField`       | General file uploads (PDFs, docs, etc.)          | `upload_to` path     |
| `AutoField`       | Auto-incrementing integer (used for primary key) | —                    |

---

### Field Examples in a Model

```python
# models.py
from django.db import models

class Employee(models.Model):
    # Text Fields
    name        = models.CharField(max_length=150)
    bio         = models.TextField()

    # Numeric Fields
    age         = models.IntegerField()

    # Validation Fields
    email       = models.EmailField()

    # Boolean Field
    is_active   = models.BooleanField(default=True)

    # Date & Time
    created_at  = models.DateTimeField(auto_now_add=True)

    # File Uploads
    profile_pic = models.ImageField(upload_to='profiles/')
    resume      = models.FileField(upload_to='resumes/')

    # AutoField (Django adds this automatically as `id`)
    # id = models.AutoField(primary_key=True)  ← implicit
```

> **Note:** Django automatically adds an `id = AutoField(primary_key=True)` to every model unless you explicitly define your own primary key.

---

## 3. The Migrations Workflow

Migrations are Django's way of **propagating changes** made to your models into the actual database schema. The workflow involves two commands used in sequence:

```
[1] Edit models.py
        ↓
[2] python manage.py makemigrations
        ↓
[3] python manage.py migrate
```

---

### Step 1 — `makemigrations`

```bash
python manage.py makemigrations
```

This command:
- Scans `models.py` for any **new or changed** model definitions
- Generates a **migration file** (e.g., `0001_initial.py`) inside the `migrations/` folder
- Acts as a **blueprint** describing what database changes need to be made

> Think of `makemigrations` as writing the plan — it does **not** touch the database yet.

---

### Step 2 — `migrate`

```bash
python manage.py migrate
```

This command:
- Reads all pending migration files
- **Applies** the changes to the actual database
- Updates the database tables to match your current Python models

> Think of `migrate` as executing the plan — this is where the database is actually modified.

---

### Migration File Location

```
your_app/
│
├── migrations/
│   ├── __init__.py
│   ├── 0001_initial.py       ← First migration (CreateModel)
│   ├── 0002_add_field.py     ← Subsequent changes (AddField, etc.)
│
├── models.py
└── views.py
```

---

## 4. Anatomy of a Migration File

Django auto-generates migration files — understanding their structure helps with debugging and advanced use.

```python
# migrations/0001_initial.py

from django.db import migrations, models

class Migration(migrations.Migration):

    # Migrations that must be applied before this one
    dependencies = []

    # The actual database operations to perform
    operations = [
        migrations.CreateModel(
            name='Employee',
            fields=[
                ('id', models.AutoField(primary_key=True)),
                ('name', models.CharField(max_length=150)),
                ('age', models.IntegerField()),
                ('email', models.EmailField()),
            ],
        ),
    ]
```

### Key Sections

| Section        | Description                                                                 |
|----------------|-----------------------------------------------------------------------------|
| `dependencies` | A list of migrations that must run first. Ensures changes happen in order.  |
| `operations`   | The specific actions to perform: `CreateModel`, `AddField`, `DeleteField`, etc. |

---

### Common Migration Operations

| Operation       | Triggered When                          |
|-----------------|-----------------------------------------|
| `CreateModel`   | A new model class is added              |
| `AddField`      | A new field is added to an existing model |
| `RemoveField`   | A field is deleted from a model         |
| `AlterField`    | A field's properties are modified       |
| `DeleteModel`   | An entire model class is removed        |

---

## 5. Important Interview Concepts

### How Django Detects Changes

When you run `migrate`, Django does **not** simply look at the last migration file. Instead, it:

1. Runs **all migration files** in order to rebuild a complete internal "state" of your models
2. Compares that state against the **current database structure**
3. Applies only the changes that are missing

This is why the order of migrations (managed via `dependencies`) is critical to maintaining consistency.

---

### Deleting Migration Files ⚠️

> **Warning:** Deleting migration files is dangerous and should be avoided in production.

Django tracks every applied migration in an internal database table called `django_migrations`. If you delete a migration file that has already been applied:

- Django loses the history it needs to track state
- Running `migrate` again will throw errors due to the inconsistency
- The `django_migrations` table and your actual file history will be out of sync

---

### Primary Key

A **Primary Key** is a unique identifier for each record in a database table. In Django:

- Every model automatically gets an `id` field of type `AutoField` as its default primary key
- It auto-increments with each new record (`1, 2, 3, ...`)
- You can define a custom primary key by setting `primary_key=True` on any field

```python
class Product(models.Model):
    sku = models.CharField(max_length=20, primary_key=True)  # Custom primary key
    name = models.CharField(max_length=100)
```

---

## 6. Database Default

Django uses **SQLite3** as its default database — a lightweight, file-based database ideal for development.

| Database       | Environment      | Notes                                          |
|----------------|------------------|------------------------------------------------|
| **SQLite3**    | Development      | Default, zero-config, stored in `db.sqlite3`   |
| **PostgreSQL** | Production       | Robust, scalable, widely used in production    |
| **MySQL**      | Production       | Popular alternative for production deployments |

---

### Changing the Database (in `settings.py`)

**Default (SQLite3):**

```python
# settings.py
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db.sqlite3',
    }
}
```

**Switching to PostgreSQL:**

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'your_db_name',
        'USER': 'your_db_user',
        'PASSWORD': 'your_password',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

---

### Complete Workflow Summary

```
[1] Define / Update your Model
    Edit the class in models.py
        ↓
[2] Create a Migration Blueprint
    python manage.py makemigrations
        ↓
[3] Apply Changes to the Database
    python manage.py migrate
        ↓
[4] Verify in the Admin Panel or Database
    python manage.py createsuperuser  →  http://127.0.0.1:8000/admin/
```

---

## Technology Stack

| Component    | Detail                                         |
|--------------|------------------------------------------------|
| **Language** | Python                                         |
| **Framework**| Django                                         |
| **Default DB**| SQLite3 (development), PostgreSQL/MySQL (production) |
| **Purpose**  | Database Schema Definition & Migration Management |

---

*For further reading, refer to the [official Django Models documentation](https://docs.djangoproject.com/en/stable/topics/db/models/) and the [Migrations documentation](https://docs.djangoproject.com/en/stable/topics/migrations/).*
