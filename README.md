# Django Comprehensive Class Exercise: Custom User Model and Permissions 🚀🎓🐍

In this exercise, you will create a Django application that covers a range of topics, including Django sessions, the admin panel, models, model forms with validators, user authentication, class-based views, and group permissions. You will build a custom user model with phone number authentication and custom permissions. Get ready to dive into Django and have some fun! 😄🎉

## Exercise Overview

1. Set up a new Django project and app
2. Create a custom user model with phone number authentication and custom permissions
3. Set up user authentication with a custom authentication backend
4. Create a custom manager for the user model
5. Define Django group permissions
6. Implement model forms with validators
7. Create views with class-based views and group permissions
8. Design templates with conditional menu visibility and login/logout buttons
9. Test your application

## Step 1: Set up a new Django project and app

1. Create a new Django project and a new app within the project. Name the app `custom_auth`.

## Step 2: Create a custom user model with phone number authentication and custom permissions

1. In your `custom_auth` app, create a new file named `models.py` if it doesn't exist.
2. Import the required modules and create a custom user model that inherits from `AbstractBaseUser` and `PermissionsMixin`. Add fields for phone number, email, and any other relevant information.

```python
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, PermissionsMixin
from django.core.validators import RegexValidator
from django.db import models

class CustomUser(AbstractBaseUser, PermissionsMixin):
    phone_regex = RegexValidator(
        regex=r'....',
        message="...."
    )
    phone_number = ...
    ...

    objects = CustomUserManager()

    USERNAME_FIELD = ...
    REQUIRED_FIELDS = [...]

```

## Step 3: Set up user authentication with a custom authentication backend

1. Create a new file named `backends.py` in the `custom_auth` app.
2. Define a custom authentication backend that authenticates users based on their phone number.
3. Add the custom authentication backend to your project's settings.

```python
# custom_auth/backends.py
from django.contrib.auth.backends import BaseBackend
from .models import CustomUser

class PhoneNumberBackend(BaseBackend):
    def authenticate(self, request, phone_number=None, password=None, **kwargs):
        try:
            ...
        except CustomUser.DoesNotExist:
            return None

    def get_user(self, user_id):
        try:
           ....
        except CustomUser.DoesNotExist:
            return None

# settings.py
AUTHENTICATION_BACKENDS = [
    ...
]
```

## Step 4: Create a custom manager for the user model

1. In `models.py`, create a custom manager for the `CustomUser` model that handles user creation.

```python
class CustomUserManager(BaseUserManager):
    def create_user(self, phone_number, first_name, last_name, password=None, **extra_fields):
        ...
        return user

    def create_superuser(self, phone_number, first_name, last_name, password=None, **extra_fields):
        # extra_fields.setdefault
        ...
        return self.create_user(phone_number, first_name, last_name, password, **extra_fields)
```

## Step 5: Define Django group permissions 🚀1. In your `models.py`, create a new model named `CustomGroup` that inherits from `Group`.
2. Add custom permissions to the `CustomGroup` model using the `permissions` field. 📝

```python
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager, Group, PermissionsMixin
from django.core.validators import RegexValidator
from django.db import models

class CustomUser(AbstractBaseUser, PermissionsMixin):
    # Fields for CustomUser model
    ...

class CustomGroup(Group):
    # Custom group permissions
    ...
        ]
```


## Step 6: Implement model forms with validators 🎓

1. Create a new file named `forms.py` in the `custom_auth` app.
2. Define a `CustomUserCreationForm` and a `CustomUserChangeForm` that inherit from `UserCreationForm` and `UserChangeForm`, respectively.
3. Add validators for the phone number field. 🐍

```python
from django import forms
from django.contrib.auth.forms import UserCreationForm, UserChangeForm
from django.core.validators import RegexValidator
from .models import CustomUser

class CustomUserCreationForm(UserCreationForm):
    ...

    class Meta(UserCreationForm.Meta):
        ...

class CustomUserChangeForm(UserChangeForm):
    ...

    class Meta(UserChangeForm.Meta):
        ...
```

## Step 7: Create views with class-based views and group permissions 📝

1. Create a new file named `views.py` in the `custom_auth` app.
2. Define class-based views that handle user authentication, user creation, and user editing.
3. Add group permissions to restrict access to certain views. 🎉

```python
from django.contrib.auth import authenticate, login
from django.contrib.auth.decorators import login_required, permission_required
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin
from django.contrib.auth.views import LoginView, LogoutView
from django.urls import reverse_lazy
from django.views.generic import CreateView, DetailView, UpdateView
from .forms import CustomUserCreationForm, CustomUserChangeForm
from .models import CustomUser

class CustomLoginView(LoginView):
    template_name = 'custom_auth/login.html'
    fields = '__all__'
    redirect_authenticated_user = True

class CustomLogoutView(LogoutView):
    template_name = 'custom_auth/logout.html'

class CustomUserCreateView(CreateView):
    model = CustomUser
    form_class = CustomUserCreationForm
    success_url = reverse_lazy('profile')
    template_name = 'custom_auth/signup.html'

class CustomUserDetailView(LoginRequiredMixin, PermissionRequiredMixin, DetailView):
    model = CustomUser
    template_name = 'custom_auth/profile.html'
    permission_required = 'auth.view_user'

class CustomUserUpdateView(LoginRequiredMixin, PermissionRequiredMixin, UpdateView):
    model = CustomUser
    form_class = CustomUserChangeForm
    success_url = reverse_lazy('profile')
    template_name = 'custom_auth/edit_profile.html'
    permission_required = 'auth.change_user'
```

## Step 8: Design templates with conditional menu visibility and login/logout buttons

1. Create a base template with a menu that is only visible to users with specific group permissions, and login/logout buttons that appear conditionally.

```html
<!-- templates/base.html -->
{% load static %}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}Django Custom Auth{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'css/main.css' %}">
</head>
<body>
    <header>
        <nav>
            {% if user.has_perm('custom_auth.view_post') %}
                <a href="/posts/">Posts</a>
            {% endif %}
            {% if user.has_perm('custom_auth.add_post') %}
                <a href="/posts/new/">New Post</a>
            {% endif %}
            {% if user.is_authenticated %}
                <a href="{% url 'logout' %}">Logout</a>
            {% else %}
                <a href="{% url 'login' %}">Login</a>
            {% endif %}
        </nav>
    </header>
    <main>
        {% block content %}{% endblock %}
    </main>
</body>
</html>
```

2. Create templates for the post list, post creation, and post deletion views.

```html
<!-- templates/custom_auth/post_list.html -->
{% extends 'base.html' %}
{% block content %}
    <h1>Posts</h1>
    <ul>
        {% for post in object_list %}
            <li>{{ post.title }}</li>
        {% endfor %}
    </ul>
{% endblock %}
```

```html
<!-- templates/custom_auth/post_form.html -->
{% extends 'base.html' %}
{% block content %}
    <h1>New Post</h1>
    <form method="post">
        {% csrf_token %}
        {{ form.as_p }}
        <button type="submit">Save</button>
    </form>
{% endblock %}
```

```html
<!-- templates/custom_auth/post_confirm_delete.html -->
{% extends 'base.html' %}
{% block content %}
    <h1>Delete Post</h1>
    <p>Are you sure you want to delete this post?</p>
    <form method="post">
        {% csrf_token %}
        <button type="submit">Yes, delete</button>
    </form>
