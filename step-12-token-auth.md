# Step 12: Add Token Authentication

## Instructions

### 1. Create Tokens for Existing Users

```bash
python manage.py shell
```

In the Django shell, run:

```python
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token

# Create tokens for all existing users
for user in User.objects.all():
    Token.objects.get_or_create(user=user)
    print(f"Token created for user: {user.username}")

exit()
```

### 2. Test Authentication with Token

Get a token for your user:

```bash
python manage.py shell
```

```python
from django.contrib.auth.models import User
from rest_framework.authtoken.models import Token

user = User.objects.get(username='testuser')
token = Token.objects.get(user=user)
print(f"Token for {user.username}: {token.key}")

exit()
```

### 3. Test Protected Endpoints

Use the token in your API requests:

```bash
# Test creating a post with authentication
curl -X POST http://127.0.0.1:8000/api/posts/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Token YOUR_TOKEN_HERE" \
  -d '{
    "title": "Authenticated Post",
    "content": "This post requires authentication",
    "published": true
  }'
```

### 4. Test Unauthorized Access

Try accessing protected endpoints without a token:

```bash
# This should return 401 Unauthorized
curl -X POST http://127.0.0.1:8000/api/posts/ \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Unauthorized Post",
    "content": "This should fail",
    "published": true
  }'
```

### 5. Test Read-Only Access

Test that unauthenticated users can still read posts:

```bash
# This should work without authentication
curl -X GET http://127.0.0.1:8000/api/posts/
```

### 6. Test Create Post (with token)

```bash
curl -X POST http://127.0.0.1:8000/api/posts/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Token YOUR_TOKEN_HERE" \
  -d '{
    "title": "My First API Post",
    "content": "This post was created via the API",
    "published": true
  }'
```

### 7. Test Update Post

```bash
curl -X PUT http://127.0.0.1:8000/api/posts/1/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Token YOUR_TOKEN_HERE" \
  -d '{
    "title": "Updated Post Title",
    "content": "This post has been updated",
    "published": true
  }'
```

### 8. Test Create Comment

```bash
curl -X POST http://127.0.0.1:8000/api/posts/1/comments/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Token YOUR_TOKEN_HERE" \
  -d '{
    "content": "Great post! Thanks for sharing."
  }'
```

### 9. Test Delete Post

```bash
curl -X DELETE http://127.0.0.1:8000/api/posts/1/ \
  -H "Authorization: Token YOUR_TOKEN_HERE"
```

### 10. Test Logout

```bash
curl -X POST http://127.0.0.1:8000/api/auth/logout/ \
  -H "Authorization: Token YOUR_TOKEN_HERE"
```

Next: [Step 13: Admin Interface](step-13-admin-interface.md)
