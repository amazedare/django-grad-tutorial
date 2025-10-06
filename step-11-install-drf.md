# Step 6: Install Django REST Framework

## Instructions

### 1. Install Django REST Framework

```bash
pip install djangorestframework
```

### 2. Update Requirements File

```bash
pip freeze > requirements.txt
```

### 4. Add DRF to your application
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',  # Django REST Framework
    'rest_framework.authtoken',  # Token authentication
    'blog',  # Add your app here
]
```

### 4. Configure Django REST Framework

Add these settings at the end of the file:

```python
# Django REST Framework
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20
}
```

Next: [Step 12: Tokan Auth](step-12-token-auth.md)
