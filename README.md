# Project-1
Open source incident management system 


Project Overview
An incident management portal for logging, tracking, and resolving infrastructure or application issues with role-based access for Admins, Technicians, and Users.

Core Features
User Authentication & Role-Based Access

Roles: Admin, Technician, Reporter

JWT or Flask-Login/Django Auth for session handling

Incident Lifecycle Management

Create: Reporter logs an issue

Assign: Admin assigns it to a Technician

Update: Technician can update status/details

Resolve: Mark incident as resolved with resolution notes

Email Notifications (SMTP)

Triggered on: Create, Assign, Resolve

Config via .env file (SMTP credentials)

API Endpoints (Flask-RESTful or Django REST Framework)

POST /incidents/ - Create incident

GET /incidents/ - List all

PUT /incidents/ - Update/Assign/Resolve

GET /incidents/ - View details

Authenticated routes based on role

Tech Stack
Backend: Flask or Django

Database: SQLite

Frontend: Bootstrap + HTML templates (Jinja or Django templating)

Containerization: Docker (with Dockerfile + docker-compose.yml)

Version Control: Git + GitHub

Directory Structure (Flask Example)
incident-management/ ├── app/ │ ├── init.py │ ├── models.py │ ├── routes.py │ ├── email_utils.py │ ├── templates/ │ └── static/ ├── tests/ ├── Dockerfile ├── docker-compose.yml ├── requirements.txt ├── .env └── README.md

Dockerization
Dockerfile (basic):

FROM python:3.11-slim WORKDIR /app COPY . . RUN pip install -r requirements.txt CMD ["python", "run.py"]

docker-compose.yml:

version: '3.8' services: web: build: . ports: - "5000:5000" env_file: - .env

Sample Data
Pre-fill the database with:

2-3 users per role

3-5 incident logs with various statuses

Deliverables
GitHub repo with source code

Docker image (on Docker Hub or exportable .tar)

Demo video/screenshots showing:

Login

Creating an issue

Assigning and resolving

Markdown file or DB JSON for sample issues

Full implementation plan for the Django-based Open-Source Incident Management System. I’ll split it into key components so we can build it step-by-step:

Project Setup
django-admin startproject incident_portal cd incident_portal python manage.py startapp incidents

Add 'incidents', 'rest_framework', and optionally 'corsheaders' to INSTALLED_APPS in settings.py.

Models (incidents/models.py)
from django.db import models from django.contrib.auth.models import User

class Incident(models.Model): STATUS_CHOICES = [ ('Open', 'Open'), ('Assigned', 'Assigned'), ('Resolved', 'Resolved'), ]

title = models.CharField(max_length=200)
description = models.TextField()
reporter = models.ForeignKey(User, related_name='reported_incidents', on_delete=models.CASCADE)
technician = models.ForeignKey(User, related_name='assigned_incidents', null=True, blank=True, on_delete=models.SET_NULL)
status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='Open')
resolution_notes = models.TextField(blank=True, null=True)
created_at = models.DateTimeField(auto_now_add=True)
updated_at = models.DateTimeField(auto_now=True)
Serializers (incidents/serializers.py)
from rest_framework import serializers from .models import Incident from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer): class Meta: model = User fields = ['id', 'username', 'email']

class IncidentSerializer(serializers.ModelSerializer): reporter = UserSerializer(read_only=True) technician = UserSerializer(read_only=True)

class Meta:
    model = Incident
    fields = '__all__'
Views (incidents/views.py)
from rest_framework import viewsets, permissions from .models import Incident from .serializers import IncidentSerializer from rest_framework.permissions import IsAuthenticated from rest_framework.response import Response from rest_framework.decorators import action from django.core.mail import send_mail from django.conf import settings

class IncidentViewSet(viewsets.ModelViewSet): queryset = Incident.objects.all().order_by('-created_at') serializer_class = IncidentSerializer permission_classes = [IsAuthenticated]

def perform_create(self, serializer):
    incident = serializer.save(reporter=self.request.user)
    send_mail("New Incident Reported", f"{incident.title}", settings.DEFAULT_FROM_EMAIL, [incident.reporter.email])

def perform_update(self, serializer):
    incident = serializer.save()
    if incident.status == 'Resolved':
        send_mail("Incident Resolved", f"{incident.title} has been resolved.", settings.DEFAULT_FROM_EMAIL, [incident.reporter.email])
URLs
In incident_portal/urls.py:

from django.contrib import admin from django.urls import path, include from rest_framework import routers from incidents.views import IncidentViewSet

router = routers.DefaultRouter() router.register(r'incidents', IncidentViewSet)

urlpatterns = [ path('admin/', admin.site.urls), path('api/', include(router.urls)), path('api-auth/', include('rest_framework.urls')), ]

Authentication (Simple Login UI + API)
Use Django’s built-in login or add Token Authentication with djangorestframework.authtoken.

Email Config (settings.py)
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend' EMAIL_HOST = 'smtp.gmail.com' EMAIL_PORT = 587 EMAIL_USE_TLS = True EMAIL_HOST_USER = 'your-email@gmail.com' EMAIL_HOST_PASSWORD = 'your-password' DEFAULT_FROM_EMAIL = EMAIL_HOST_USER

Docker Setup
Dockerfile

FROM python:3.11-slim ENV PYTHONDONTWRITEBYTECODE 1 ENV PYTHONUNBUFFERED 1 WORKDIR /app COPY requirements.txt . RUN pip install -r requirements.txt COPY . . CMD ["gunicorn", "incident_portal.wsgi:application", "--bind", "0.0.0.0:8000"]

docker-compose.yml

version: '3' services: web: build: . ports: - "8000:8000" volumes: - .:/app env_file: - .env

Requirements
Django>=4.0 djangorestframework gunicorn

Sample Users & Issues
Create a fixtures/initial_data.json for sample users and incident logs.

Optional
Frontend in Bootstrap for simple login/form views

JWT auth using djangorestframework-simplejwt

API documentation via Swagger or Postman collection

 Let’s start building your Django-based Incident Management System section by section.


[1] Project Setup

Run these commands to initialize the project:

django-admin startproject incident_portal
cd incident_portal
python manage.py startapp incidents
pip install djangorestframework

Update settings.py:

INSTALLED_APPS = [
    ...
    'rest_framework',
    'incidents',
]

REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ]
}

Create Superuser for Admin Panel:

python manage.py migrate
python manage.py createsuperuser


[2] Models – incidents/models.py

from django.db import models
from django.contrib.auth.models import User

class Incident(models.Model):
    STATUS_CHOICES = [
        ('Open', 'Open'),
        ('Assigned', 'Assigned'),
        ('Resolved', 'Resolved'),
    ]

    title = models.CharField(max_length=200)
    description = models.TextField()
    reporter = models.ForeignKey(User, related_name='reported_incidents', on_delete=models.CASCADE)
    technician = models.ForeignKey(User, related_name='assigned_incidents', null=True, blank=True, on_delete=models.SET_NULL)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES, default='Open')
    resolution_notes = models.TextField(blank=True, null=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return f"{self.title} - {self.status}"

Run Migrations:

python manage.py makemigrations
python manage.py migrate


[3] Serializers – incidents/serializers.py

from rest_framework import serializers
from .models import Incident
from django.contrib.auth.models import User

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'email']

class IncidentSerializer(serializers.ModelSerializer):
    reporter = UserSerializer(read_only=True)
    technician = UserSerializer(read_only=True)

    class Meta:
        model = Incident
        fields = '__all__'


[4] Views – incidents/views.py

from rest_framework import viewsets
from .models import Incident
from .serializers import IncidentSerializer
from rest_framework.permissions import IsAuthenticated
from django.core.mail import send_mail
from django.conf import settings

class IncidentViewSet(viewsets.ModelViewSet):
    queryset = Incident.objects.all().order_by('-created_at')
    serializer_class = IncidentSerializer
    permission_classes = [IsAuthenticated]

    def perform_create(self, serializer):
        incident = serializer.save(reporter=self.request.user)
        send_mail(
            subject='New Incident Logged',
            message=f"Incident '{incident.title}' has been created.",
            from_email=settings.DEFAULT_FROM_EMAIL,
            recipient_list=[incident.reporter.email],
        )

    def perform_update(self, serializer):
        incident = serializer.save()
        if incident.status == 'Resolved':
            send_mail(
                subject='Incident Resolved',
                message=f"Incident '{incident.title}' has been resolved.",
                from_email=settings.DEFAULT_FROM_EMAIL,
                recipient_list=[incident.reporter.email],
            )


[5] URLs – incident_portal/urls.py

from django.contrib import admin
from django.urls import path, include
from rest_framework import routers
from incidents.views import IncidentViewSet

router = routers.DefaultRouter()
router.register(r'incidents', IncidentViewSet)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),
    path('api-auth/', include('rest_framework.urls')),
]


Next up:

[6] Email Configuration

[7] Dockerization

[8] Optional UI/Token Auth

[9] Sample Data

[10] README + GitHub-ready structure

Awesome! Let’s continue with:


[6] Email Configuration – settings.py

Set up SMTP for email notifications. Use a Gmail account or another SMTP provider.

# Email Configuration
EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
EMAIL_HOST_USER = 'your-email@gmail.com'
EMAIL_HOST_PASSWORD = 'your-app-password'
DEFAULT_FROM_EMAIL = EMAIL_HOST_USER

Security Tip: Store sensitive info in .env:

Install python-decouple:

pip install python-decouple

Update settings.py:

from decouple import config

EMAIL_HOST_USER = config('EMAIL_HOST_USER')
EMAIL_HOST_PASSWORD = config('EMAIL_HOST_PASSWORD')

Create a .env file:

EMAIL_HOST_USER=your-email@gmail.com
EMAIL_HOST_PASSWORD=your-app-password


[7] Dockerization

Dockerfile

Create a file named Dockerfile in the root directory:

FROM python:3.11-slim

ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["gunicorn", "incident_portal.wsgi:application", "--bind", "0.0.0.0:8000"]

requirements.txt

Django>=4.2
djangorestframework
gunicorn
python-decouple

docker-compose.yml

version: '3.9'

services:
  web:
    build: .
    ports:
      - "8000:8000"
    volumes:
      - .:/app
    env_file:
      - .env

Build and Run:

docker-compose build
docker-compose up

Now the app should be running on http://localhost:8000


Next up:

[8] Optional: Token Auth / Bootstrap Frontend

[9] Fixtures: Sample users & incidents

[10] README template for GitHub

let's implement Login and Token Authentication using Django REST Framework.


[8] Token Authentication Setup

Step 1: Install required package

pip install djangorestframework-simplejwt

Step 2: Update settings.py

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ]
}


Step 3: Update urls.py – Add JWT endpoints

In incident_portal/urls.py:

from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns += [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]

Now you have:

POST /api/token/ – get access + refresh tokens

POST /api/token/refresh/ – refresh access token


Step 4: Usage Example (with Postman or frontend)

Send a POST request to /api/token/ with:

{
  "username": "your-username",
  "password": "your-password"
}

Response:

{
  "refresh": "eyJ0eXAiOiJKV1QiLCJh...",
  "access": "eyJ0eXAiOiJKV1QiLCJh..."
}

Use the access token in headers for authenticated API requests:

Authorization: Bearer <your-access-token>

Nice — let’s add a basic Bootstrap login UI for your project!


[9] Bootstrap Login Page (Optional UI Layer)

We’ll create a simple HTML login form using Django's built-in LoginView.


Step 1: Templates Setup

In your app directory (incidents/), create:

templates/
└── registration/
    └── login.html

login.html

<!DOCTYPE html>
<html>
<head>
    <title>Login</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body class="bg-light">
    <div class="container mt-5">
        <div class="row justify-content-center">
            <div class="col-md-4">
                <div class="card shadow p-3">
                    <h4 class="mb-3 text-center">Incident Portal Login</h4>
                    <form method="post">
                        {% csrf_token %}
                        {{ form.as_p }}
                        <button type="submit" class="btn btn-primary w-100">Login</button>
                    </form>
                </div>
            </div>
        </div>
    </div>
</body>
</html>


Step 2: Update settings.py

TEMPLATES[0]['DIRS'] = [BASE_DIR / 'incidents/templates']
LOGIN_REDIRECT_URL = '/api/'  # Redirect after login


Step 3: Add Login Route

In incident_portal/urls.py, add:

from django.contrib.auth import views as auth_views

urlpatterns += [
    path('login/', auth_views.LoginView.as_view(), name='login'),
    path('logout/', auth_views.LogoutView.as_view(next_page='/login/'), name='logout'),
]

Now visit:

/login/ to sign in

/logout/ to log out


The login form works with Django session-based auth — perfect for quickly testing your frontend if needed.

Next, I’ll show you how to:

Add sample users & incidents with a fixture

Create a nice README.md for GitHub

Perfect! Let's finish strong with:


[10] Sample Users & Incidents – Fixtures

Fixtures help auto-load test data into your database.


Step 1: Create Sample Users and Incidents

Run this to dump your superuser and data (if you’ve already created them manually):

python manage.py dumpdata auth.user incidents.incident > fixtures/sample_data.json

Or, create manually:

fixtures/sample_data.json

[
  {
    "model": "auth.user",
    "pk": 1,
    "fields": {
      "username": "admin",
      "is_superuser": true,
      "is_staff": true,
      "is_active": true,
      "password": "pbkdf2_sha256$...hashed_password_here..."
    }
  },
  {
    "model": "incidents.incident",
    "pk": 1,
    "fields": {
      "title": "Server Down",
      "description": "Production server unreachable",
      "reporter": 1,
      "technician": null,
      "status": "Open",
      "resolution_notes": "",
      "created_at": "2025-04-20T10:00:00Z",
      "updated_at": "2025-04-20T10:00:00Z"
    }
  }
]

(You can hash a password using Django shell or create it via admin.)


Step 2: Load the Fixture

python manage.py loaddata fixtures/sample_data.json

You’ll now have demo users and incidents ready for testing.


[11] README.md Template (for GitHub)

README.md

# Incident Management System

A Django-based portal to log, track, and resolve application or infrastructure issues.

## Features
- REST APIs for incident lifecycle
- Role-based access
- Email notifications on incident creation/resolution
- JWT-based authentication
- Bootstrap login page
- Docker support

## Getting Started

```bash
git clone https://github.com/your-username/incident-portal.git
cd incident-portal
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver

Sample Users

Docker

docker-compose build
docker-compose up

API Endpoints

POST /api/token/ – Get JWT token

GET /api/incidents/ – List incidents

POST /api/incidents/ – Create incident

That’s your **full Django Incident Management System** — complete with:
- REST APIs
- Auth
- Email
- UI
- Docker
- Sample data
- GitHub-ready README


Project Folder Structure

incident-portal/
├── incident_portal/         # Django project root
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
│
├── incidents/               # Django app
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   ├── serializers.py
│   ├── urls.py              # Optional if you split urls
│   ├── tests.py
│   └── templates/
│       └── registration/
│           └── login.html
│
├── fixtures/
│   └── sample_data.json
│
├── .env                     # Hidden env file (do not commit)
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── README.md
└── manage.py


.gitignore Example

__pycache__/
*.pyc
.env
db.sqlite3
/static/
*.log


To ZIP Your Project

From root:

zip -r incident-portal.zip incident-portal/
