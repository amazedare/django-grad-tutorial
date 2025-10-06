# Step 11: Test API with curl

## Instructions

### 1. Start the Development Server

```bash
python manage.py runserver
```

### 2. Test User Registration

```bash
curl -X POST http://127.0.0.1:8000/api/auth/register/ \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "testpass123",
    "email": "test@example.com"
  }'
```

### 3. Test User Login

```bash
curl -X POST http://127.0.0.1:8000/api/auth/login/ \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "testpass123"
  }'
```


### 5. Test Get All Posts

```bash
curl -X GET http://127.0.0.1:8000/api/posts/
```

### 6. Test Get Specific Post

```bash
curl -X GET http://127.0.0.1:8000/api/posts/1/
```

Save the token from the response for the next steps.



### 9. Test Get Comments for Post

```bash
curl -X GET http://127.0.0.1:8000/api/posts/1/comments/
```

Next: [Step 12: Add Token Authentication](step-12-token-auth.md)
