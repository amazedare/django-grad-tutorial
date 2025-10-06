# Step 14: Add Privacy Controls (Extension)

## Instructions

### 1. Update Models for Privacy

```bash
code blog/models.py
```

Add these fields to the Post model:

```python
# blog/models.py
from django.db import models
from django.contrib.auth.models import User
from django.urls import reverse
from django.utils import timezone

class Post(models.Model):
    """Blog post model"""
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    published = models.BooleanField(default=False)
    
    # Privacy controls
    PRIVACY_CHOICES = [
        ('public', 'Public - Anyone can see and comment'),
        ('followers', 'Followers only - Only followers can see and comment'),
        ('private', 'Private - Only specific users can see and comment'),
    ]
    privacy_level = models.CharField(max_length=20, choices=PRIVACY_CHOICES, default='public')
    allowed_users = models.ManyToManyField(User, related_name='allowed_posts', blank=True)
    
    class Meta:
        ordering = ['-created_at']
    
    def __str__(self):
        return self.title
    
    def get_absolute_url(self):
        return reverse('post-detail', kwargs={'pk': self.pk})
    
    def can_user_see(self, user):
        """Check if a user can see this post"""
        if not self.published:
            return False
        
        if self.privacy_level == 'public':
            return True
        elif self.privacy_level == 'followers':
            # For simplicity, we'll consider all users as "followers" for now
            # In a real app, you'd have a Follow model
            return user.is_authenticated
        elif self.privacy_level == 'private':
            return user in self.allowed_users.all() or user == self.author
        
        return False
    
    def can_user_comment(self, user):
        """Check if a user can comment on this post"""
        return self.can_user_see(user) and user.is_authenticated

class Comment(models.Model):
    """Comment model for blog posts"""
    post = models.ForeignKey(Post, on_delete=models.CASCADE, related_name='comments')
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    approved = models.BooleanField(default=False)
    
    class Meta:
        ordering = ['created_at']
    
    def __str__(self):
        return f'Comment by {self.author.username} on {self.post.title}'
```

### 2. Create and Apply Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

### 3. Update Serializers

```bash
code blog/serializers.py
```

Update the PostSerializer:

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
    allowed_users = UserSerializer(many=True, read_only=True)
    can_see = serializers.SerializerMethodField()
    can_comment = serializers.SerializerMethodField()
    
    class Meta:
        model = Post
        fields = [
            'id', 'title', 'content', 'author', 'created_at', 'updated_at', 
            'published', 'privacy_level', 'allowed_users', 'can_see', 'can_comment'
        ]
        read_only_fields = ['id', 'author', 'created_at', 'updated_at', 'allowed_users', 'can_see', 'can_comment']
    
    def get_can_see(self, obj):
        request = self.context.get('request')
        if request and request.user.is_authenticated:
            return obj.can_user_see(request.user)
        return obj.privacy_level == 'public'
    
    def get_can_comment(self, obj):
        request = self.context.get('request')
        if request and request.user.is_authenticated:
            return obj.can_user_comment(request.user)
        return False

class PostCreateUpdateSerializer(serializers.ModelSerializer):
    allowed_user_ids = serializers.ListField(
        child=serializers.IntegerField(),
        write_only=True,
        required=False
    )
    
    class Meta:
        model = Post
        fields = [
            'title', 'content', 'published', 'privacy_level', 'allowed_user_ids'
        ]
    
    def create(self, validated_data):
        allowed_user_ids = validated_data.pop('allowed_user_ids', [])
        post = Post.objects.create(**validated_data)
        
        if allowed_user_ids:
            allowed_users = User.objects.filter(id__in=allowed_user_ids)
            post.allowed_users.set(allowed_users)
        
        return post
    
    def update(self, instance, validated_data):
        allowed_user_ids = validated_data.pop('allowed_user_ids', [])
        
        for attr, value in validated_data.items():
            setattr(instance, attr, value)
        instance.save()
        
        if allowed_user_ids is not None:
            allowed_users = User.objects.filter(id__in=allowed_user_ids)
            instance.allowed_users.set(allowed_users)
        
        return instance

class CommentSerializer(serializers.ModelSerializer):
    author = UserSerializer(read_only=True)
    post = serializers.PrimaryKeyRelatedField(read_only=True)
    
    class Meta:
        model = Comment
        fields = ['id', 'content', 'author', 'post', 'created_at', 'approved']
        read_only_fields = ['id', 'author', 'post', 'created_at']
```

### 4. Update API Views

```bash
code blog/views.py
```

Update the views to handle privacy:

```python
# blog/views.py
from rest_framework import generics, permissions, status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.response import Response
from rest_framework.authtoken.models import Token
from django.contrib.auth import authenticate
from django.contrib.auth.models import User
from django.db.models import Q
from .models import Post, Comment
from .serializers import (
    PostSerializer, PostCreateUpdateSerializer, CommentSerializer, UserSerializer
)

class PostListCreateView(generics.ListCreateAPIView):
    serializer_class = PostSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    def get_queryset(self):
        user = self.request.user
        if user.is_authenticated:
            # Show posts the user can see
            return Post.objects.filter(
                Q(privacy_level='public') |
                Q(privacy_level='followers') |
                Q(privacy_level='private', allowed_users=user) |
                Q(author=user)
            ).distinct()
        else:
            # Only show public posts for anonymous users
            return Post.objects.filter(privacy_level='public')
    
    def get_serializer_class(self):
        if self.request.method == 'POST':
            return PostCreateUpdateSerializer
        return PostSerializer
    
    def perform_create(self, serializer):
        serializer.save(author=self.request.user)

class PostDetailView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Post.objects.all()
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    def get_serializer_class(self):
        if self.request.method in ['PUT', 'PATCH']:
            return PostCreateUpdateSerializer
        return PostSerializer
    
    def get_queryset(self):
        user = self.request.user
        if user.is_authenticated:
            return Post.objects.filter(
                Q(privacy_level='public') |
                Q(privacy_level='followers') |
                Q(privacy_level='private', allowed_users=user) |
                Q(author=user)
            ).distinct()
        else:
            return Post.objects.filter(privacy_level='public')
    
    def get_permissions(self):
        if self.request.method in ['PUT', 'PATCH', 'DELETE']:
            return [permissions.IsAuthenticated()]
        return [permissions.AllowAny()]

class CommentListCreateView(generics.ListCreateAPIView):
    serializer_class = CommentSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    def get_queryset(self):
        post_id = self.kwargs['post_id']
        try:
            post = Post.objects.get(id=post_id)
            if post.can_user_see(self.request.user):
                return Comment.objects.filter(post_id=post_id, approved=True)
        except Post.DoesNotExist:
            pass
        return Comment.objects.none()
    
    def perform_create(self, serializer):
        post_id = self.kwargs['post_id']
        post = Post.objects.get(id=post_id)
        
        if not post.can_user_comment(self.request.user):
            raise permissions.PermissionDenied("You don't have permission to comment on this post")
        
        serializer.save(author=self.request.user, post=post)

class CommentDetailView(generics.RetrieveUpdateDestroyAPIView):
    queryset = Comment.objects.all()
    serializer_class = CommentSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]
    
    def get_permissions(self):
        if self.request.method in ['PUT', 'PATCH', 'DELETE']:
            return [permissions.IsAuthenticated()]
        return [permissions.AllowAny()]

@api_view(['GET'])
@permission_classes([permissions.IsAuthenticated])
def user_list(request):
    """Get list of users for privacy settings"""
    users = User.objects.exclude(id=request.user.id)
    serializer = UserSerializer(users, many=True)
    return Response(serializer.data)

@api_view(['POST'])
@permission_classes([permissions.AllowAny])
def register_user(request):
    username = request.data.get('username')
    password = request.data.get('password')
    email = request.data.get('email', '')
    
    if not username or not password:
        return Response({'error': 'Username and password are required'}, 
                       status=status.HTTP_400_BAD_REQUEST)
    
    if User.objects.filter(username=username).exists():
        return Response({'error': 'Username already exists'}, 
                       status=status.HTTP_400_BAD_REQUEST)
    
    user = User.objects.create_user(username=username, password=password, email=email)
    token, created = Token.objects.get_or_create(user=user)
    
    return Response({
        'token': token.key,
        'user': UserSerializer(user).data
    }, status=status.HTTP_201_CREATED)

@api_view(['POST'])
@permission_classes([permissions.AllowAny])
def login_user(request):
    username = request.data.get('username')
    password = request.data.get('password')
    
    if not username or not password:
        return Response({'error': 'Username and password are required'}, 
                       status=status.HTTP_400_BAD_REQUEST)
    
    user = authenticate(username=username, password=password)
    
    if user:
        token, created = Token.objects.get_or_create(user=user)
        return Response({
            'token': token.key,
            'user': UserSerializer(user).data
        })
    else:
        return Response({'error': 'Invalid credentials'}, 
                       status=status.HTTP_401_UNAUTHORIZED)

@api_view(['POST'])
@permission_classes([permissions.IsAuthenticated])
def logout_user(request):
    try:
        request.user.auth_token.delete()
        return Response({'message': 'Successfully logged out'})
    except:
        return Response({'error': 'Error logging out'}, 
                       status=status.HTTP_400_BAD_REQUEST)
```

### 5. Update URLs

```bash
code blog/urls.py
```

Add the new endpoint:

```python
# blog/urls.py
from django.urls import path
from . import views

urlpatterns = [
    # Posts
    path('posts/', views.PostListCreateView.as_view(), name='post-list-create'),
    path('posts/<int:pk>/', views.PostDetailView.as_view(), name='post-detail'),
    
    # Comments
    path('posts/<int:post_id>/comments/', views.CommentListCreateView.as_view(), name='comment-list-create'),
    path('comments/<int:pk>/', views.CommentDetailView.as_view(), name='comment-detail'),
    
    # Users
    path('users/', views.user_list, name='user-list'),
    
    # Authentication
    path('auth/register/', views.register_user, name='register'),
    path('auth/login/', views.login_user, name='login'),
    path('auth/logout/', views.logout_user, name='logout'),
]
```

### 6. Test Privacy Controls

```bash
# Start the server
python manage.py runserver
```

#### Test Public Post (anyone can see)

```bash
# Create a public post
curl -X POST http://127.0.0.1:8000/api/posts/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Token YOUR_TOKEN_HERE" \
  -d '{
    "title": "Public Post",
    "content": "This is a public post",
    "published": true,
    "privacy_level": "public"
  }'
```

#### Test Private Post (specific users only)

```bash
# Get list of users first
curl -X GET http://127.0.0.1:8000/api/users/ \
  -H "Authorization: Token YOUR_TOKEN_HERE"

# Create a private post (replace USER_ID with actual user ID)
curl -X POST http://127.0.0.1:8000/api/posts/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Token YOUR_TOKEN_HERE" \
  -d '{
    "title": "Private Post",
    "content": "This is a private post",
    "published": true,
    "privacy_level": "private",
    "allowed_user_ids": [USER_ID]
  }'
```

#### Test Access Control

```bash
# Try to access posts without authentication (should only see public posts)
curl -X GET http://127.0.0.1:8000/api/posts/

# Access posts with authentication (should see all allowed posts)
curl -X GET http://127.0.0.1:8000/api/posts/ \
  -H "Authorization: Token YOUR_TOKEN_HERE"
```

#### Test Comment Permissions

```bash
# Try to comment on a private post (should fail if not allowed)
curl -X POST http://127.0.0.1:8000/api/posts/2/comments/ \
  -H "Content-Type: application/json" \
  -H "Authorization: Token YOUR_TOKEN_HERE" \
  -d '{
    "content": "This comment might not be allowed"
  }'
```

## Complete!

Your Django REST API now includes:
- **Privacy levels**: Public, Followers only, Private
- **User-specific access control**: Users can specify who can see their posts
- **Comment permissions**: Users can only comment on posts they can see
- **User management**: API endpoint to get list of users for privacy settings
- **Access control**: Proper filtering based on privacy settings and user permissions
