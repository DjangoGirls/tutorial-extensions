# Homework: create comment model

Now we only have Post model, what about receiving some feedback from your readers?

## Creating comment blog model

Let's open `blog/models.py` and append this piece of code to the end of file:

    class Comment(models.Model):
    	post = models.ForeignKey('blog.Post')
    	author = models.CharField(max_length=200)
    	text = models.TextField()
    	created_date = models.DateTimeField(default=timezone.now)
    	approved = models.BooleanField(default=False)

    	def approve(self):
    		self.approved = True
    		self.save()

    	def __str__(self):
    		return self.text

You can go back to **Django models** chapter in tutorial if you need to remind yourself what each of field types means.

In this chapter we have new type of field:
- `models.BooleanField` - this is true/false field.


## Create tables for models in your database

Now it's time to add our comment model to database. To do this we have to let know Django that we made changes in our model. Type `python manage.py makemigrations blog`. Just like this:

    (myvenv) ~/djangogirls$ python manage.py makemigrations blog
    Migrations for 'blog':
      0002_comment.py:
        - Create model Comment

You can see that this command created for us another migration file in `blog/migrations/` directory. Now we need to apply those changes with `python manage.py migrate blog`. It should look like this:

    (myvenv) ~/djangogirls$ python manage.py migrate blog
    Operations to perform:
      Apply all migrations: blog
    Running migrations:
      Rendering model states... DONE
      Applying blog.0002_comment... OK

Our Comment model exists in database now. It would be nice if we had access to it in our admin panel.










