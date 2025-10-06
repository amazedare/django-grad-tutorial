# Step 1: Project Setup

## Instructions

### 1. Create Project Directory

```bash
mkdir django-blog-project
cd django-blog-project
```

### 2. Create Virtual Environment

```bash
python -m venv venv
```

### 3. Activate Virtual Environment

**Windows:**
```bash
venv\Scripts\activate
```

**macOS/Linux:**
```bash
source venv/bin/activate
```

### 4. Install Django

```bash
pip install django
python -m django --version
```

### 5. Create Requirements File

```bash
pip freeze > requirements.txt
```

Next: [Step 2: Create Django Project](step-02-create-project.md)
