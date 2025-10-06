# Step 4: Configure Settings

## Instructions

### 1. Open Settings File

```bash
code blogs/settings.py
```

### 2. Add Blog App to INSTALLED_APPS

Find the `INSTALLED_APPS` section and add `'blog'`:

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',  # Add your app here
]
```

Next: [Step 5: Create Models](step-05-create-models.md)
