# Step 13: Admin Interface

## Instructions

### 1. Configure Admin for Your Models

```bash
code blog/admin.py
```

Replace the contents of `blog/admin.py` with:

```python
# blog/admin.py
from django.contrib import admin
from .models import Post, Comment

@admin.register(Post)
class PostAdmin(admin.ModelAdmin):
    list_display = ['title', 'author', 'published', 'created_at', 'updated_at']
    list_filter = ['published', 'created_at', 'author']
    search_fields = ['title', 'content', 'author__username']
    list_editable = ['published']
    date_hierarchy = 'created_at'
    ordering = ['-created_at']
    
    fieldsets = (
        ('Post Information', {
            'fields': ('title', 'content', 'author')
        }),
        ('Publication', {
            'fields': ('published',)
        }),
        ('Timestamps', {
            'fields': ('created_at', 'updated_at'),
            'classes': ('collapse',)
        }),
    )
    
    readonly_fields = ['created_at', 'updated_at']
    
    def get_queryset(self, request):
        return super().get_queryset(request).select_related('author')

@admin.register(Comment)
class CommentAdmin(admin.ModelAdmin):
    list_display = ['content', 'author', 'post', 'approved', 'created_at']
    list_filter = ['approved', 'created_at', 'author']
    search_fields = ['content', 'author__username', 'post__title']
    list_editable = ['approved']
    date_hierarchy = 'created_at'
    ordering = ['-created_at']
    
    fieldsets = (
        ('Comment Information', {
            'fields': ('content', 'author', 'post')
        }),
        ('Moderation', {
            'fields': ('approved',)
        }),
        ('Timestamps', {
            'fields': ('created_at',),
            'classes': ('collapse',)
        }),
    )
    
    readonly_fields = ['created_at']
    
    def get_queryset(self, request):
        return super().get_queryset(request).select_related('author', 'post')
```

### 2. Access the Admin Interface

1. Make sure your Django server is running:
   ```bash
   python manage.py runserver
   ```

2. Open your browser and go to: `http://127.0.0.1:8000/admin/`

3. Login with your superuser credentials

### 3. Explore the Admin Interface

You should see the Django admin dashboard with:
- **Users**: Manage user accounts and tokens
- **Groups**: Manage user groups
- **Posts**: Manage blog posts
- **Comments**: Manage comments
- **Tokens**: View and manage API tokens

### 4. Manage API Tokens

1. Click on **"Tokens"** under the **"Authentication and Authorization"** section
2. You can:
   - View all API tokens
   - Delete tokens
   - See which user each token belongs to

### 5. Manage Blog Posts

1. Click on **"Posts"** under the **"Blog"** section
2. You can:
   - Add new posts
   - Edit existing posts
   - Delete posts
   - Use search and filter options

### 6. Manage Comments

1. Click on **"Comments"** under the **"Blog"** section
2. You can:
   - Approve comments
   - Edit comment content
   - Delete inappropriate comments

## Complete!

Your Django REST API is now complete with:
- Token-based authentication
- RESTful API endpoints
- User registration and login
- Blog post CRUD operations
- Comment system
- Admin interface for content management
- API token management

Next: [Step 14: Privacy Controls](step-14-privacy-controls.md)