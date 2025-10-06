# Step 9: Configure URLs

## Instructions

### 1. Create App URLs File

```bash
touch blog/urls.py
code blog/urls.py
```

### 2. Configure Blog App URLs

Add the following content to `blog/urls.py`:

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
    
    # Authentication
    path('auth/register/', views.register_user, name='register'),
    path('auth/login/', views.login_user, name='login'),
    path('auth/logout/', views.logout_user, name='logout'),
]
```

### 3. Configure Main Project URLs

```bash
code blogs/urls.py
```

Replace the contents of `blogs/urls.py` with:

```python
# blogs/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('blog.urls')),
]
```

Next: [Step 10: Run Migrations](step-10-run-migrations.md)
