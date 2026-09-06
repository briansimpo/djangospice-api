# Djangospice API

Reusable API utilities for **Django** and **Django REST Framework**.

Djangospice API provides reusable serializers, API routing utilities, permissions, and common ViewSet functionality for building consistent and modular Django REST APIs.

---

## Features

* Automatic API router discovery
* Modular application-level API routers
* Central API routing
* Automatic model-based ViewSet registration
* OpenAPI/Swagger tags
* Default API model permissions
* Action-aware serializers
* Nested relationship serialization
* Relationship ID and nested writes
* Generic related-object serialization
* Standard option serializers
* Bulk deletion support

---

## Requirements

* Python 3.12+
* Django 5.0+
* Django REST Framework 3.15+

---

## Installation

```bash
pip install djangospice-api
```

---

## Configuration

Add the package to your Django project:

```python
INSTALLED_APPS = [
    # ...
    "djangospice_api",
]
```

Include the API URLs:

```python
from django.urls import include, path


urlpatterns = [
    path("api/", include("djangospice_api.urls")),
]
```

Once configured, Djangospice API can discover API routers provided by installed applications automatically.

---

# API Routers

Applications can expose their API endpoints through:

```text
<application>/api/router.py
```

For example:

```text
courses/
└── api/
    └── router.py
```

The router can be a standard Django REST Framework router:

```python
from rest_framework.routers import DefaultRouter

from .views import CourseViewSet


router = DefaultRouter()

router.register(
    "courses",
    CourseViewSet,
    basename="course",
)
```

The application's routes are automatically included in the project's API.

For example:

```text
/api/courses/
/api/courses/{id}/
```

Applications do not need to depend on the Djangospice runtime to define their routers.

---

# APIRouter

Djangospice API provides `APIRouter` for applications that need additional routing functionality.

```python
from djangospice_api.router import APIRouter
```

It supports:

* model-based registration
* automatic ViewSet registration
* router composition
* default API permissions
* OpenAPI tags

A shared router instance is also provided:

```python
from djangospice_api.router import router
```

---

## Registering a Model ViewSet

A ViewSet can be registered using an application label and model name:

```python
router.register_model(
    "students",
    "student",
    StudentViewSet,
)
```

A custom basename can be provided:

```python
router.register_model(
    "students",
    "student",
    StudentViewSet,
    basename="students",
)
```

---

## Automatic ViewSet Registration

For conventional ModelViewSets, the model can be detected from the ViewSet's queryset:

```python
router.register_viewset(
    StudentViewSet,
)
```

For example:

```python
class StudentViewSet(ModelViewSet):
    queryset = Student.objects.all()
    serializer_class = StudentSerializer
```

The API router derives the application and model information automatically.

---

# API Permissions

Djangospice API provides a default model-based permission system.

When a ViewSet does not define its own permission classes, the API router can apply:

```python
ApiModelPermissions
```

Custom permissions can be supplied during registration:

```python
router.register_viewset(
    StudentViewSet,
    permission_classes=[
        IsAuthenticated,
    ],
)
```

Explicit permissions defined by a ViewSet are preserved unless registration specifies an override.

---

# OpenAPI Documentation

When ViewSets are registered automatically, Djangospice API can provide OpenAPI tags for generated API documentation.

For example, application-based registration can produce tags such as:

```text
Students
Courses
Billing
```

Custom tags can also be supplied:

```python
router.register_viewset(
    StudentViewSet,
    swagger_tags=[
        "Academic",
        "Students",
    ],
)
```

This integrates with `drf-spectacular`.

---

# BaseModelSerializer

`BaseModelSerializer` extends Django REST Framework's `ModelSerializer` with action-specific representations.

```python
from djangospice_api.serializers import BaseModelSerializer
```

Example:

```python
class CourseSerializer(BaseModelSerializer):

    class Meta:
        model = Course

        fields = [
            "id",
            "code",
            "name",
            "department",
            "description",
        ]

        list_fields = [
            "id",
            "code",
            "name",
            "department",
        ]

        detail_fields = [
            "id",
            "code",
            "name",
            "department",
            "description",
        ]
```

Different field configurations can be provided for:

* list
* detail
* create
* update
* partial update

When an action-specific configuration is not supplied, the standard `fields` configuration is used.

---

# Flattening Relationships

Relationships can be represented as human-readable values in list responses.

```python
class CourseSerializer(BaseModelSerializer):

    class Meta:
        model = Course

        fields = [
            "id",
            "code",
            "name",
            "department",
        ]

        flatten_fields = [
            "department",
        ]
```

A list response can then represent the department using its string representation rather than only its primary key.

This is useful for:

* data tables
* administrative interfaces
* lookup interfaces
* HTMX applications
* JavaScript clients

---

# Hidden List Fields

Fields can be excluded from list responses:

```python
class Meta:
    model = Course

    fields = [
        "id",
        "code",
        "name",
        "description",
        "created_at",
    ]

    hidden_fields = [
        "description",
        "created_at",
    ]
```

The fields remain available to other serializer actions.

---

# GenericObjectRelatedField

`GenericObjectRelatedField` provides a generic representation of related objects.

```python
from djangospice_api.serializers import GenericObjectRelatedField
```

A related object is represented using:

```json
{
    "id": 42,
    "model_type": "Course",
    "display_name": "Database Systems"
}
```

This is useful when an API works with generic or polymorphic relationships.

---

# NestedModelSerializer

`NestedModelSerializer` extends `BaseModelSerializer` with support for Django model relationships.

```python
from djangospice_api.serializers import NestedModelSerializer
```

It supports:

* ForeignKey
* OneToOneField
* ManyToManyField

Nested serializers can be discovered using conventional serializer names or explicitly configured.

Example:

```python
class CourseSerializer(NestedModelSerializer):

    class Meta:
        model = Course

        fields = [
            "id",
            "code",
            "name",
            "department",
        ]
```

---

# Nested Serializer Configuration

Nested serializers can be explicitly specified:

```python
class CourseSerializer(NestedModelSerializer):

    class Meta:
        model = Course

        fields = [
            "id",
            "code",
            "name",
            "department",
        ]

        serializers = {
            "department": DepartmentSerializer,
        }
```

This is useful when a model has multiple possible serializers.

---

# Relationship Writes

Djangospice API provides consistent conventions for relationship writes.

| Relationship      | Write field    |
| ----------------- | -------------- |
| ForeignKey        | `field_id`     |
| OneToOne          | `field_id`     |
| ManyToMany        | `field_ids`    |
| Nested ForeignKey | `field_nested` |
| Nested OneToOne   | `field_nested` |
| Nested ManyToMany | `field_nested` |

For example:

```json
{
    "name": "Advanced Databases",
    "prerequisites_ids": [
        10,
        11,
        12
    ]
}
```

Nested relationships can also be supplied:

```json
{
    "name": "Advanced Databases",
    "prerequisites_nested": [
        {
            "id": 10
        },
        {
            "name": "Data Structures"
        }
    ]
}
```

---

# OptionModelSerializer

`OptionModelSerializer` provides a standard serializer for option and reference models.

```python
from djangospice_api.serializers import OptionModelSerializer
```

The standard fields are:

```text
id
name
value
slug
code
description
is_visible
```

Example:

```python
class StatusSerializer(OptionModelSerializer):

    class Meta:
        model = Status
```

This is useful for:

* dropdown options
* lookup values
* status definitions
* configuration data
* reference data

---

# BulkDeleteMixin

`BulkDeleteMixin` adds bulk deletion support to Django REST Framework ViewSets.

```python
from djangospice_api.mixins import BulkDeleteMixin
```

Example:

```python
class CourseViewSet(
    BulkDeleteMixin,
    ModelViewSet,
):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer
```

The ViewSet provides:

```text
POST /courses/bulk-delete/
```

Request:

```json
{
    "ids": [
        1,
        2,
        3
    ]
}
```

Response:

```json
{
    "deleted": 3
}
```

---

# Example

A typical Django application can combine the utilities:

```python
from rest_framework.routers import DefaultRouter
from rest_framework.viewsets import ModelViewSet

from djangospice_api.mixins import BulkDeleteMixin

from .models import Course
from .serializers import CourseSerializer


class CourseViewSet(
    BulkDeleteMixin,
    ModelViewSet,
):
    queryset = Course.objects.all()
    serializer_class = CourseSerializer


router = DefaultRouter()

router.register(
    "courses",
    CourseViewSet,
    basename="course",
)
```

Place the router in:

```text
courses/api/router.py
```

and it will be discovered automatically.

---

# Development

Clone the repository:

```bash
git clone https://github.com/briansimpo/djangospice_api.git
cd djangospice_api
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate the environment and install the package:

```bash
pip install -e .
```

---

# License

djangospice-api is licensed under the **MIT License**.

See [LICENSE](LICENSE) for the full license text.

---

# Project

[Djangospice API on GitHub](https://github.com/briansimpo/djangospice_api?utm_source=chatgpt.com)

