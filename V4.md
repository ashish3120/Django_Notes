# Django Template Engine – Dynamic Rendering Guide

> The Django Template Engine enables dynamic content rendering within HTML, allowing views to pass live data, apply logic, and share layout structure across pages — all without duplicating code.

---

## Table of Contents

1. [Dynamic Data & Context](#1-dynamic-data--context)
2. [Template Logic (Tags)](#2-template-logic-tags)
3. [Template Filters](#3-template-filters)
4. [Template Inheritance (DRY Principle)](#4-template-inheritance-dry-principle)
5. [Summary of Syntax](#5-summary-of-syntax)

---

## 1. Dynamic Data & Context

### Static vs. Dynamic Data

| Type             | Description                                                   |
|------------------|---------------------------------------------------------------|
| **Static Data**  | Fixed content — the same for every user on every request      |
| **Dynamic Data** | Comes from the server or database and changes per user/request |

---

### The Context Dictionary

To send data from a **View** to a **Template**, you pass a Python dictionary as the `context` argument inside `render()`.

```python
# views.py
from django.shortcuts import render

def home(request):
    people_list = [
        {'name': 'Alice', 'age': 25},
        {'name': 'Bob', 'age': 16},
    ]
    return render(request, "index.html", context={'peoples': people_list})
```

> The keys of the `context` dictionary become the **variable names** accessible inside the template.

---

### Accessing Variables in Templates

Use **double curly braces** `{{ }}` to print a variable's value inside your HTML:

```html
<!-- index.html -->
<p>{{ peoples }}</p>
```

Accessing a specific attribute of an object:

```html
<p>{{ person.name }}</p>
<p>{{ person.age }}</p>
```

---

## 2. Template Logic (Tags)

Django uses special **template tags** `{% %}` to handle logic such as loops and conditions directly inside HTML.

---

### For Loops

Iterate through a list passed via the context dictionary:

```html
{% for person in peoples %}
    <p>{{ person.name }}</p>
{% endfor %}
```

> Always close every loop with `{% endfor %}`.

**Built-in Loop Variable — `forloop.counter`:**

`forloop.counter` provides the current iteration index, starting from `1`:

```html
{% for person in peoples %}
    <p>{{ forloop.counter }}. {{ person.name }}</p>
{% endfor %}
```

**Output example:**
```
1. Alice
2. Bob
3. Charlie
```

---

### If / Else Conditions

Use `{% if %}` for conditional rendering:

```html
{% if person.age >= 18 %}
    <p>{{ person.name }} is an adult.</p>
{% else %}
    <p>{{ person.name }} is a minor.</p>
{% endif %}
```

> Always close with `{% endif %}`. The `{% else %}` block is optional.

---

### Logical Operators

You can use Python-like logical operators inside template tags:

| Operator   | Example Usage                              |
|------------|--------------------------------------------|
| `and`      | `{% if user.active and user.verified %}`   |
| `or`       | `{% if user.admin or user.staff %}`        |
| `in`       | `{% if name in approved_list %}`           |
| `not in`   | `{% if name not in banned_list %}`         |

```html
{% if person.age >= 18 and person.name in approved_list %}
    <p>Access granted.</p>
{% endif %}
```

---

## 3. Template Filters

Filters modify how variables are **displayed** using the pipe `|` symbol. They do not alter the actual data — only its presentation.

**Syntax:**

```html
{{ variable|filter_name }}
{{ variable|filter_name:argument }}
```

---

### Common Built-in Filters

| Filter           | Syntax                          | Description                                      |
|------------------|---------------------------------|--------------------------------------------------|
| `truncatechars`  | `{{ text\|truncatechars:10 }}`  | Limits output to a set number of characters      |
| `first`          | `{{ list\|first }}`             | Returns the first item of a list                 |
| `upper`          | `{{ name\|upper }}`             | Converts string to UPPERCASE                     |
| `lower`          | `{{ name\|lower }}`             | Converts string to lowercase                     |

**Examples:**

```html
<!-- Truncate to 10 characters -->
<p>{{ article.body|truncatechars:10 }}</p>

<!-- Get first item from a list -->
<p>{{ peoples|first }}</p>

<!-- Change casing -->
<p>{{ person.name|upper }}</p>
<p>{{ person.name|lower }}</p>
```

> For a full list of built-in filters, refer to the [official Django Template Filter documentation](https://docs.djangoproject.com/en/stable/ref/templates/builtins/#built-in-filter-reference).

---

## 4. Template Inheritance (DRY Principle)

### The Problem

Without inheritance, common HTML sections like headers, footers, and navigation bars would need to be **copy-pasted** into every template — making updates tedious and error-prone.

### The Solution — Template Inheritance

Django's inheritance system allows you to define a **base template** once and have child templates extend it, injecting only their unique content.

> **DRY** = *Don't Repeat Yourself* — a core principle of clean software development.

---

### Step 1 — Create `base.html`

Define the shared page structure with `{% block %}` placeholders where child content will be injected:

```html
<!-- templates/base.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>My Django App</title>
</head>
<body>

    <nav>
        <a href="/">Home</a> | <a href="/about">About</a>
    </nav>

    <!-- Placeholder for child content -->
    {% block start %}
    {% endblock %}

    <footer>
        <p>&copy; 2025 My App</p>
    </footer>

</body>
</html>
```

---

### Step 2 — Extend the Base in a Child Template

Use `{% extends %}` at the **very top** of the child template, then fill in the defined blocks:

```html
<!-- templates/home/index.html -->
{% extends "base.html" %}

{% block start %}
    <h1>Welcome to the Home Page</h1>

    {% for person in peoples %}
        <p>{{ forloop.counter }}. {{ person.name }} — Age: {{ person.age }}</p>
    {% endfor %}
{% endblock %}
```

> `{% extends %}` must always be the **first line** in a child template.

---

### Inheritance Flow

```
base.html                    child template (index.html)
─────────────────────        ──────────────────────────────
<nav> ... </nav>             {% extends "base.html" %}
{% block start %}      ←     {% block start %}
{% endblock %}               <h1>Unique content here</h1>
<footer> ... </footer>       {% endblock %}
```

Multiple blocks can be defined for different regions (e.g., sidebar, scripts, styles):

```html
{% block title %}Page Title{% endblock %}
{% block content %}Main content{% endblock %}
{% block scripts %}Extra JS here{% endblock %}
```

---

## 5. Summary of Syntax

| Type         | Syntax                    | Purpose                                   |
|--------------|---------------------------|-------------------------------------------|
| **Variables**| `{{ variable_name }}`     | Print data/values from the context        |
| **Tags**     | `{% tag_logic %}`         | Loops, conditions, and inheritance        |
| **Filters**  | `{{ value\|filter }}`     | Modify or format how data is displayed    |
| **Comments** | `{# comment #}`           | Internal template notes (not rendered)    |

---

### Complete Example

**`views.py`**

```python
from django.shortcuts import render

def home(request):
    people_list = [
        {'name': 'Alice', 'age': 25},
        {'name': 'Bob', 'age': 16},
    ]
    return render(request, "home/index.html", {'peoples': people_list})
```

**`templates/base.html`**

```html
<!DOCTYPE html>
<html>
<body>
    <nav>Site Navigation</nav>
    {% block start %}{% endblock %}
    <footer>Site Footer</footer>
</body>
</html>
```

**`templates/home/index.html`**

```html
{% extends "base.html" %}

{% block start %}
    <h1>People List</h1>
    {% for person in peoples %}
        <p>
            {{ forloop.counter }}.
            {{ person.name|upper }} —
            {% if person.age >= 18 %}
                Adult
            {% else %}
                Minor
            {% endif %}
        </p>
    {% endfor %}
{% endblock %}
```

---

## Technology Stack

| Component    | Detail                                            |
|--------------|---------------------------------------------------|
| **Language** | Python, HTML                                      |
| **Framework**| Django                                            |
| **Purpose**  | Dynamic Content Rendering & Template Reusability  |

---

*For a full list of built-in tags and filters, refer to the [official Django Template documentation](https://docs.djangoproject.com/en/stable/ref/templates/builtins/).*
