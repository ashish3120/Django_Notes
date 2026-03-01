# Django CRUD Operations – Complete Database Interaction Guide

> CRUD operations are the fundamental building blocks of any web application's database interaction. Django's ORM makes it possible to perform all four operations using clean, readable Python — no raw SQL required.

---

## Table of Contents

1. [What is CRUD?](#1-what-is-crud)
2. [Create (Adding Data)](#2-create-adding-data)
3. [Read (Fetching Data)](#3-read-fetching-data)
4. [Update (Modifying Data)](#4-update-modifying-data)
5. [Delete (Removing Data)](#5-delete-removing-data)
6. [Enhanced Readability with `__str__`](#6-enhanced-readability-with-__str__)
7. [Complete CRUD Reference](#7-complete-crud-reference)

---

## 1. What is CRUD?

CRUD describes the four essential operations every application needs to interact with a database:

| Letter | Operation  | Description                               |
|--------|------------|-------------------------------------------|
| **C**  | **Create** | Add new records to the database           |
| **R**  | **Read**   | Retrieve existing records from the database |
| **U**  | **Update** | Modify existing data in the database      |
| **D**  | **Delete** | Remove records from the database          |

---

### Sample Model Used in This Guide

All examples use the following `Car` model:

```python
# models.py
from django.db import models

class Car(models.Model):
    car_name = models.CharField(max_length=100)
    speed    = models.IntegerField()

    def __str__(self) -> str:
        return self.car_name
```

> Remember to run `python manage.py makemigrations` and `python manage.py migrate` after defining your model.

---

## 2. Create (Adding Data)

Django provides three ways to create and save a new record to the database:

---

### Method 1 — Instantiate and Save Manually

```python
c = Car(car_name="Nexon", speed=110)
c.save()  # Must call save() to commit to the database
```

**When to use:** When you need to modify or validate the object before it's saved, or trigger custom logic between instantiation and saving.

---

### Method 2 — `objects.create()` ✅ Recommended

```python
Car.objects.create(car_name="Fortuner", speed=80)
# Automatically saves — no need to call .save()
```

**When to use:** For straightforward, single-step record creation with no pre-save manipulation needed.

---

### Method 3 — Unpacking a Dictionary

```python
car_data = {'car_name': 'XUV700', 'speed': 150}
Car.objects.create(**car_data)
# Useful when data comes from a form, API, or dynamic source
```

**When to use:** When your data is already structured as a dictionary (e.g., from a form submission or JSON payload). The `**` operator unpacks the dictionary into keyword arguments.

---

### Create Methods Compared

| Method                    | Auto-saves | Best For                                     |
|---------------------------|------------|----------------------------------------------|
| `Car(...)` + `.save()`    | ❌ No      | Pre-save logic or conditional saving         |
| `Car.objects.create(…)`   | ✅ Yes     | Simple, direct record creation               |
| `Car.objects.create(**d)` | ✅ Yes     | Dynamic or dictionary-based data input       |

---

## 3. Read (Fetching Data)

Django's ORM provides several methods for retrieving records, each suited to different scenarios:

---

### Fetch All Records

```python
Car.objects.all()
# Returns a QuerySet containing all Car records
```

---

### Get a Single Record — `.get()`

```python
car = Car.objects.get(id=1)
print(car.car_name)  # Output: Nexon
```

> ⚠️ **Warning:** If no record matches the given condition, `.get()` raises a `DoesNotExist` exception — which will crash your application if unhandled.

**Safe usage with error handling:**

```python
try:
    car = Car.objects.get(id=99)
except Car.DoesNotExist:
    print("Record not found.")
```

---

### Filter Records — `.filter()`

```python
cars = Car.objects.filter(id=10)
# Returns a QuerySet — empty [] if no match, never raises an exception
```

> ✅ **Benefit:** `.filter()` always returns a QuerySet. If no records match, it returns an **empty QuerySet** `[]` instead of raising an error — making it safer for general use.

---

### `.get()` vs. `.filter()` — Key Differences

| Feature              | `.get()`                            | `.filter()`                          |
|----------------------|-------------------------------------|--------------------------------------|
| **Returns**          | A single model instance             | A QuerySet (list-like)               |
| **No match found**   | ❌ Raises `DoesNotExist` exception   | ✅ Returns empty QuerySet `[]`        |
| **Multiple matches** | ❌ Raises `MultipleObjectsReturned` | ✅ Returns all matching records       |
| **Best for**         | Fetching one specific record safely | Filtering or searching across records |

---

## 4. Update (Modifying Data)

Django provides two approaches to updating records — one for individual objects and one for bulk operations:

---

### Method 1 — Fetch, Modify, and Save

```python
# Step 1: Fetch the record
car = Car.objects.get(id=1)

# Step 2: Modify the field
car.car_name = "Creta"

# Step 3: Save the change
car.save()
```

**When to use:** When you need to update a single record or perform validation/logic before saving.

---

### Method 2 — Bulk Update with `.update()` ✅ Efficient

```python
Car.objects.filter(id=1).update(car_name="Creta Dark Edition")
# Directly updates the database — no need to fetch the object first
```

**When to use:** When updating one or more records that match a filter condition. This is more efficient as it hits the database directly without fetching the object into memory.

---

### Update Methods Compared

| Method                        | Fetches Object | SQL Equivalent         | Best For                         |
|-------------------------------|----------------|------------------------|----------------------------------|
| Fetch → Modify → `.save()`    | ✅ Yes         | `SELECT` then `UPDATE` | Single record, pre-save logic    |
| `.filter().update(…)`         | ❌ No          | Direct `UPDATE`        | Bulk or direct field updates     |

---

## 5. Delete (Removing Data)

> ⚠️ **Deletion is permanent.** Always double-check your filter conditions before deleting, especially for bulk operations.

---

### Delete a Specific Record

```python
Car.objects.get(id=1).delete()
# Fetches the record with id=1 and deletes it
```

---

### Bulk Delete — Delete All Records

```python
Car.objects.all().delete()
# ⚠️ WARNING: Deletes EVERY record in the Car table
```

---

### Safer Bulk Delete Using a Filter

```python
Car.objects.filter(speed__lt=50).delete()
# Deletes only records where speed is less than 50
```

---

### Delete Operations Summary

| Command                          | What Gets Deleted                        |
|----------------------------------|------------------------------------------|
| `Car.objects.get(id=1).delete()` | Single record with id = 1                |
| `Car.objects.filter(…).delete()` | All records matching the filter condition |
| `Car.objects.all().delete()`     | ⚠️ Every record in the table             |

---

## 6. Enhanced Readability with `__str__`

By default, Django represents model instances as generic strings like `<Car: Car object (1)>` in the shell and admin panel — which is not useful for debugging.

### The Problem

```python
>>> Car.objects.all()
<QuerySet [<Car: Car object (1)>, <Car: Car object (2)>, <Car: Car object (3)>]>
```

### The Solution — Define `__str__`

Add a `__str__` method to your model to return a human-readable representation:

```python
# models.py
class Car(models.Model):
    car_name = models.CharField(max_length=100)
    speed    = models.IntegerField()

    def __str__(self) -> str:
        return self.car_name
```

### After Adding `__str__`

```python
>>> Car.objects.all()
<QuerySet [<Car: Nexon>, <Car: Fortuner>, <Car: XUV700>]>
```

> `__str__` improves readability in the **Django Shell**, the **Admin Panel**, and anywhere model instances are printed or logged.

---

## 7. Complete CRUD Reference

### Quick Command Reference

| Operation        | Command                                              | Notes                                         |
|------------------|------------------------------------------------------|-----------------------------------------------|
| **Create**       | `Car.objects.create(car_name="X", speed=100)`        | Auto-saves                                    |
| **Create (dict)**| `Car.objects.create(**car_data)`                     | Unpack dictionary                             |
| **Read All**     | `Car.objects.all()`                                  | Returns QuerySet                              |
| **Read One**     | `Car.objects.get(id=1)`                              | Raises error if not found                     |
| **Filter**       | `Car.objects.filter(speed=110)`                      | Safe — returns empty QuerySet if no match     |
| **Update (obj)** | `car.car_name = "X"` → `car.save()`                 | Fetch first, then modify and save             |
| **Update (bulk)**| `Car.objects.filter(id=1).update(car_name="X")`      | Direct DB update, no fetch needed             |
| **Delete (one)** | `Car.objects.get(id=1).delete()`                     | Deletes one record                            |
| **Delete (all)** | `Car.objects.all().delete()`                         | ⚠️ Deletes every record                       |

---

### Complete Shell Session Example

```python
# python manage.py shell

from home.models import Car

# ── CREATE ──────────────────────────────────────────
Car.objects.create(car_name="Nexon", speed=110)
Car.objects.create(car_name="Fortuner", speed=80)

car_data = {'car_name': 'XUV700', 'speed': 150}
Car.objects.create(**car_data)

# ── READ ─────────────────────────────────────────────
all_cars = Car.objects.all()
print(all_cars)               # <QuerySet [<Car: Nexon>, ...]>

one_car = Car.objects.get(id=1)
print(one_car.car_name)       # Nexon

filtered = Car.objects.filter(speed=110)
print(filtered)               # <QuerySet [<Car: Nexon>]>

# ── UPDATE ───────────────────────────────────────────
car = Car.objects.get(id=1)
car.car_name = "Creta"
car.save()

Car.objects.filter(id=2).update(car_name="Creta Dark Edition")

# ── DELETE ───────────────────────────────────────────
Car.objects.get(id=1).delete()        # Delete one
Car.objects.all().delete()            # ⚠️ Delete all

exit()
```

---

### Workflow Tip

> CRUD operations are typically performed inside `views.py` in a real application. However, testing them first in the **Django Shell** (`python manage.py shell`) is the best way to verify your query logic before wiring it into views.

---

## Technology Stack

| Component     | Detail                                              |
|---------------|-----------------------------------------------------|
| **Language**  | Python                                              |
| **Framework** | Django                                              |
| **Interface** | Django ORM & Django Shell                           |
| **Purpose**   | Full Database CRUD Operations via Python            |

---

*For further reading, refer to the [official Django QuerySet API documentation](https://docs.djangoproject.com/en/stable/ref/models/querysets/).*
