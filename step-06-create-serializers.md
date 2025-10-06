# Step 7: Create Serializers

## Instructions

### 1. Create Serializers File

```bash
touch blog/serializers.py
code blog/serializers.py
```

### 2. Create the Serializers

Add the following content to `blog/serializers.py`:

```python
# blog/serializers.py
from rest_framework import serializers
from django.contrib.auth.models import User
from .models import Post, Comment

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email', 'first_name', 'last_name']
        read_only_fields = ['id']

class PostSerializer(serializers.ModelSerializer):
    author = UserSerializer(read_only=True)
    
    class Meta:
        model = Post
        fields = ['id', 'title', 'content', 'author', 'created_at', 'updated_at', 'published']
        read_only_fields = ['id', 'author', 'created_at', 'updated_at']

class CommentSerializer(serializers.ModelSerializer):
    author = UserSerializer(read_only=True)
    post = serializers.PrimaryKeyRelatedField(read_only=True)
    
    class Meta:
        model = Comment
        fields = ['id', 'content', 'author', 'post', 'created_at', 'approved']
        read_only_fields = ['id', 'author', 'post', 'created_at']
```

Next: [Step 7: Create API Views](step-07-create-api-views.md)
