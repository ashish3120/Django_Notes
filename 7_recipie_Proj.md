# Django RecipeProject – Part 1: Models, Forms & Data Submission

> A hands-on project walkthrough demonstrating how to build a real-world Django application. Part 1 covers setting up the model, creating the frontend form, handling POST requests, and redirecting after submission.

---

## Table of Contents

1. [Project Setup & App Creation](#1-project-setup--app-creation)
2. [Defining the Model (`models.py`)](#2-defining-the-model-modelspy)
3. [Building the Frontend Form](#3-building-the-frontend-form)
4. [Handling POST Requests in Views](#4-handling-post-requests-in-views)
5. [Redirecting After Submission](#5-redirecting-after-submission)
6. [Key Takeaways](#6-key-takeaways)

---

## 1. Project Setup & App Creation

### Create the App

```bash
python manage.py startapp vege
```

### Register the App

Open `settings.py` and add `'vege'` to the `INSTALLED_APPS` list:

```python
# settings.py
INSTALLED_APPS = [
    # Default Django apps
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',

    # Internal Apps
    'vege',
]
```

---

## 2. Defining the Model (`models.py`)

Create a `Receipe` model inside `vege/models.py` to store recipe data in the database:

```python
# vege/models.py
from django.db import models

class Receipe(models.Model):
    receipe_name        = models.CharField(max_length=200)
    receipe_description = models.TextField()
    receipe_image       = models.ImageField(upload_to='recipes/')

    def __str__(self):
        return self.receipe_name
```

### Model Fields Explained

| Field                  | Type          | Purpose                                  |
|------------------------|---------------|------------------------------------------|
| `receipe_name`         | `CharField`   | Stores the title/name of the recipe      |
| `receipe_description`  | `TextField`   | Stores the full recipe instructions      |
| `receipe_image`        | `ImageField`  | Stores the uploaded food image file path |

---

### Run Migrations

After defining the model, apply it to the database:

```bash
python manage.py makemigrations
python manage.py migrate
```

> **Note:** `ImageField` requires the `Pillow` library. Install it if not already present:
> ```bash
> pip install Pillow
> ```

---

## 3. Building the Frontend Form

### Template Setup

Create a `recipes.html` file inside `vege/templates/vege/` using Bootstrap for a clean, styled UI.

---

### Critical Form Attributes

A form that handles **file/image uploads** requires three essential elements:

```html
<!-- vege/templates/vege/recipes.html -->
<form method="POST" enctype="multipart/form-data">
    {% csrf_token %}

    <input type="text" name="receipe_name" placeholder="Recipe Name">
    <textarea name="receipe_description" placeholder="Recipe Description"></textarea>
    <input type="file" name="receipe_image">

    <button type="submit">Submit Recipe</button>
</form>
```

---

### Form Attributes Explained

| Attribute                       | Required | Purpose                                                                     |
|---------------------------------|----------|-----------------------------------------------------------------------------|
| `method="POST"`                 | ✅ Yes   | Sends data securely in the request body (not exposed in the URL)            |
| `enctype="multipart/form-data"` | ✅ Yes   | **Required for file/image uploads.** Without this, files are not transmitted |
| `{% csrf_token %}`              | ✅ Yes   | Django's security token to prevent Cross-Site Request Forgery (CSRF) attacks |
| `name="..."` on each input      | ✅ Yes   | Identifies each field so Django can retrieve the data via `request.POST`     |

> ⚠️ **Common Mistake:** Forgetting `enctype="multipart/form-data"` is the most frequent reason image uploads silently fail. The form appears to submit correctly, but no file reaches the server.

---

## 4. Handling POST Requests in Views

### Full View Logic

```python
# vege/views.py
from django.shortcuts import render, redirect
from .models import Receipe

def recipes_view(request):

    if request.method == "POST":
        # Retrieve text data
        receipe_name        = request.POST.get('receipe_name')
        receipe_description = request.POST.get('receipe_description')

        # Retrieve file/image data
        receipe_image       = request.FILES.get('receipe_image')

        # Save to the database
        Receipe.objects.create(
            receipe_name        = receipe_name,
            receipe_description = receipe_description,
            receipe_image       = receipe_image
        )

        # Redirect to prevent duplicate submissions
        return redirect('/recipes/')

    return render(request, 'vege/recipes.html')
```

---

### Step-by-Step Breakdown

**Step 1 — Check the Request Method**

```python
if request.method == "POST":
```

Only process and save data when the form has been submitted. This prevents the save logic from running on a regular page load (`GET` request).

---

**Step 2 — Retrieve Text Data**

```python
receipe_name        = request.POST.get('receipe_name')
receipe_description = request.POST.get('receipe_description')
```

`request.POST` is a dictionary-like object containing all **text fields** submitted by the form. Use `.get('name_attribute')` where the string matches the `name` attribute in your HTML input.

---

**Step 3 — Retrieve File Data**

```python
receipe_image = request.FILES.get('receipe_image')
```

`request.FILES` is a **separate** dictionary-like object that handles **uploaded files and images**. It is only populated when `enctype="multipart/form-data"` is set on the form.

---

**Step 4 — Save to the Database**

```python
Receipe.objects.create(
    receipe_name        = receipe_name,
    receipe_description = receipe_description,
    receipe_image       = receipe_image
)
```

---

### `request.POST` vs `request.FILES`

| Object           | Contains              | Used For                        |
|------------------|-----------------------|---------------------------------|
| `request.POST`   | Text, numbers, emails | All standard `<input>` fields   |
| `request.FILES`  | Files and images      | `<input type="file">` only      |

---

## 5. Redirecting After Submission

### The Problem — Duplicate Submissions

If you render the same page directly after a POST request:

```python
# ❌ Problematic approach
return render(request, 'vege/recipes.html')
```

Refreshing the browser after submission will **re-send the POST request**, creating duplicate records in the database. This is a common and frustrating bug in form handling.

---

### The Solution — Post/Redirect/Get (PRG) Pattern

```python
# ✅ Correct approach
return redirect('/recipes/')
```

After saving the data, immediately **redirect** the user to a fresh page. When they refresh, the browser re-sends a harmless `GET` request instead of repeating the `POST`.

---

### Redirect Flow Diagram

```
User Submits Form (POST)
        ↓
Django saves data to database
        ↓
Django returns redirect (302)
        ↓
Browser follows redirect (GET /recipes/)
        ↓
Page loads cleanly — refreshing is now safe ✅
```

---

## 6. Key Takeaways

| Concept                          | Rule                                                                        |
|----------------------------------|-----------------------------------------------------------------------------|
| `request.POST`                   | Use for **text fields** (name, description, email, etc.)                    |
| `request.FILES`                  | Use for **file and image uploads**                                          |
| `enctype="multipart/form-data"`  | **Mandatory** on any form that includes a file upload input                 |
| `{% csrf_token %}`               | **Always required** inside Django POST forms for security                   |
| `redirect()` after POST          | **Prevents duplicate submissions** when the user refreshes the page         |
| `name` attribute on inputs       | Required so Django can identify and retrieve each field from `request.POST` |

---

### Full Workflow Summary

```
[1] Create the app and register in settings.py
        ↓
[2] Define the model in models.py and run migrations
        ↓
[3] Build the HTML form with POST, enctype, and csrf_token
        ↓
[4] Handle POST in views.py using request.POST and request.FILES
        ↓
[5] Save with Model.objects.create()
        ↓
[6] Redirect to prevent duplicate submissions
```

---

## Technology Stack

| Component     | Detail                                          |
|---------------|-------------------------------------------------|
| **Language**  | Python, HTML                                    |
| **Framework** | Django                                          |
| **Frontend**  | Bootstrap                                       |
| **Database**  | SQLite3 (default)                               |
| **Purpose**   | Form Handling, File Uploads & Database Persistence |

---

*Next: Part 2 will cover fetching and displaying stored recipes on the frontend.*