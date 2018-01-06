# MongoDB installation

MongoDB is a schema free database that supports both relational data of SQL databases and nested datastructure of NoSQL databases. The instructions for installing MongoDB on different platforms are documented in the [MongoDB community edition manual](https://docs.mongodb.com/manual/administration/install-community/)

# Using Django with MongoDB

Every time a query is made, Django creates a SQL query string and sends it to the database. MongoDB however, uses python **query documents** to talk with the database. Pymongo is the python driver for connecting with MongoDB. A connector is needed for translating the SQL query strings into pymongo query documents.  The [Pymongo documentation](https://api.mongodb.com/python/current/tools.html) recommends using [Djongo](https://nesdis.github.io/djongo/) for connecting Django with MongoDB.

## Installing Djongo

1. pip install djongo
2. Into `settings.py` file of your project, add:

      ```python
      DATABASES = {
          'default': {
              'ENGINE': 'djongo',
              'NAME': 'your-db-name',
          }
      }
      ```
  
3. Run `manage.py makemigrations <app_name>` followed by `manage.py migrate` (ONLY the first time to create collections in MongoDB)
4. YOUR ARE SET! HAVE FUN!

## Requirements

1. Python 3.6 or higher.
2. MongoDB 3.4 or higher.
3. If your models use nested queries or sub querysets like:
  
      ```python
      inner_qs = Blog.objects.filter(name__contains='Ch').values('name')
      entries = Entry.objects.filter(blog__name__in=inner_qs)
      ```
     MongoDB 3.6 or higher is required.

# Use Django admin to add documents

The Django Admin interface can be used to add and modify documents in MongoDB. Let’s say you want to create a blogging platform using Django with MongoDB as your backend.

In your Blog `app/models.py` file define the `BlogContent` model:

```python
from djongo import models
from djongo.models import forms

class BlogContent(models.Model):
    comment = models.CharField(max_length=100)
    author = models.CharField(max_length=100)
    class Meta:
        abstract = True
```

To access the model using Django Admin you will need a Form definition for the above model. Define it as below:

```python
class BlogContentForm(forms.ModelForm):

    class Meta:
        model = BlogContent
        fields = (
            'comment', 'author'
        )
```

Now ‘embed’ your `BlogContent` inside a `BlogPost` using the `EmbeddedModelField`:

```python
class BlogPost(models.Model):
    h1 = models.CharField(max_length=100)
    content = models.EmbeddedModelField(
        model_container=BlogContent,
        model_form_class=BlogContentForm
    )   
```

That’s it you are set! Fire up Django Admin on http://localhost:8000/admin/
