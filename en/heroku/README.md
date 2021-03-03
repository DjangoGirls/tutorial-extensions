# Deploy to Heroku (as well as PythonAnywhere)

It's always good for a developer to have a couple of different deployment options under their belt.   Why not try deploying your site to Heroku, as well as PythonAnywhere?

[Heroku](http://heroku.com/) is also free for small applications that don't have too many visitors, but it's a bit more tricky to get deployed.

We will be following this tutorial: https://devcenter.heroku.com/articles/getting-started-with-django, but we pasted it here so it's easier for you.


## The `requirements.txt` file

If you didn't create one before, we need to create a `requirements.txt` file to tell Heroku what Python packages need to be installed on our server.

But first, Heroku needs us to install a few new packages. Go to your console with `virtualenv` activated and type this:

    (myvenv) $ pip install dj-database-url gunicorn whitenoise

After the installation is finished, go to the `djangogirls` directory and run this command:

    (myvenv) $ pip freeze > requirements.txt

This will create a file called `requirements.txt` with a list of your installed packages (i.e. Python libraries that you are using, for example Django :)).

> __Note__: `pip freeze` outputs a list of all the Python libraries installed in your virtualenv, and the `>` takes the output of `pip freeze` and puts it into a file. Try running `pip freeze` without the `> requirements.txt` to see what happens!

Open this file and add the following line at the bottom:

    psycopg2==2.7.2

This line is needed for your application to work on Heroku.


## Procfile

Another thing Heroku wants is a Procfile. This tells Heroku which commands to run in order to start our website. Open up your code editor, create a file called `Procfile` in `djangogirls` directory and add this line:

    web: gunicorn mysite.wsgi --log-file -

This line means that we're going to be deploying a `web` application, and we'll do that by running the command `gunicorn mysite.wsgi` (`gunicorn` is a program that's like a more powerful version of Django's `runserver` command).

Then save it. Done!

## The `runtime.txt` file

We also need to tell Heroku which Python version we want to use. This is done by creating a `runtime.txt` in the `djangogirls` directory using your editor's "new file" command, and putting the following text (and nothing else!) inside:

    python-3.6.4

## `mysite/local_settings.py`

Because it's more restrictive than PythonAnywhere, Heroku wants to use different settings from the ones we use on our locally (on our computer). Heroku wants to use Postgres while we use SQLite for example. That's why we need to create a separate file for settings that will only be available for our local environment.

Go ahead and create `mysite/local_settings.py` file. It should contain your `DATABASE` setup from your `mysite/settings.py` file. Just like that:

```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(__file__))

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

DEBUG = True
```

Then just save it! :)

## mysite/settings.py

Another thing we need to do is modify our website's `settings.py` file. Open `mysite/settings.py` in your editor and change/add the following lines:

```python
import dj_database_url

...

DEBUG = False

ALLOWED_HOSTS = ['127.0.0.1', '.herokuapp.com']

...

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'djangogirls',
        'USER': 'name',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '',
    }
}

...

db_from_env = dj_database_url.config(conn_max_age=500)
DATABASES['default'].update(db_from_env)
```

It'll do necessary configuration for Heroku.

Then save the file.

## mysite/wsgi.py

Open the `mysite/wsgi.py` file and add these lines at the end:

```python
from whitenoise.django import DjangoWhiteNoise
application = DjangoWhiteNoise(application)
```

All right!

## Heroku account

You need to install your Heroku toolbelt which you can find here (you can skip the installation if you've already installed it during setup): https://toolbelt.heroku.com/

> When running the Heroku toolbelt installation program on Windows make sure to choose "Custom Installation" when being asked which components to install. In the list of components that shows up after that please additionally check the checkbox in front of "Git and SSH".

> On Windows you also must run the following command to add Git and SSH to your command prompt's `PATH`: `setx PATH "%PATH%;C:\Program Files\Git\bin"`. Restart the command prompt program afterwards to enable the change.

> After restarting your command prompt, don't forget to go to your `djangogirls` folder again and activate your virtualenv! (Hint: [Check the Django installation chapter](http://tutorial.djangogirls.org/en/django_installation/index.html#working-with-virtualenv))

Please also create a free Heroku account here: https://id.heroku.com/signup/www-home-top

Then authenticate your Heroku account on your computer by running this command:

    $ heroku login

In case you don't have an SSH key this command will automatically create one. SSH keys are required to push code to the Heroku.

## Git commit

Heroku uses git for its deployments.  Unlike PythonAnywhere, you can push to Heroku directly, without going via Github.  But we need to tweak a couple of things first.

Open the file named `.gitignore` in your `djangogirls` directory and add `local_settings.py` to it.  We want git to ignore `local_settings`, so it stays on our local computer and doesn't end up on Heroku.

    *.pyc
    db.sqlite3
    myvenv
    __pycache__
    local_settings.py


And we commit our changes

    $ git status
    $ git add -A .
    $ git commit -m "additional files and changes for Heroku"


## Pick an application name

We'll be making your blog available on the Web at `[your blog's name].herokuapp.com`, so we need to choose a name that nobody else has taken. This name doesn't need to be related to the Django `blog` app or to `mysite` or anything we've created so far. The name can be anything you want, but Heroku is quite strict as to what characters you can use: you're only allowed to use simple lowercase letters (no capital letters or accents), numbers, and dashes (`-`).

Once you've thought of a name (maybe something with your name or nickname in it), run this command, replacing `djangogirlsblog` with your own application name:

    $ heroku create djangogirlsblog

> __Note__: Remember to replace `djangogirlsblog` with the name of your application on Heroku.

If you can't think of a name, you can instead run

    $ heroku create

and Heroku will pick an unused name for you (probably something like `enigmatic-cove-2527`).

If you ever feel like changing the name of your Heroku application, you can do so at any time with this command (replace `the-new-name` with the new name you want to use):

    $ heroku apps:rename the-new-name

> __Note__: Remember that after you change your application's name, you'll need to visit `[the-new-name].herokuapp.com` to see your site.

## Deploy to Heroku!

That was a lot of configuration and installing, right? But you only need to do that once! Now you can deploy!

When you ran `heroku create`, it automatically added the Heroku remote for our app to our repository. Now we can do a simple git push to deploy our application:

    $ git push heroku master

> __Note__: This will probably produce a *lot* of output the first time you run it, as Heroku compiles and installs psycopg. You'll know it's succeeded if you see something like `https://yourapplicationname.herokuapp.com/ deployed to Heroku` near the end of the output.

## Visit your application

You’ve deployed your code to Heroku, and specified the process types in a `Procfile` (we chose a `web` process type earlier).
We can now tell Heroku to start this `web process`.

To do that, run the following command:

    $ heroku ps:scale web=1

This tells Heroku to run just one instance of our `web` process. Since our blog application is quite simple, we don't need too much power and so it's fine to run just one process. It's possible to ask Heroku to run more processes (by the way, Heroku calls these processes "Dynos" so don't be surprised if you see this term) but it will no longer be free.

We can now visit the app in our browser with `heroku open`.

    $ heroku open

> __Note__: you will see an error page! We'll talk about that in a minute.

This will open a url like [https://djangogirlsblog.herokuapp.com/]() in your browser, and at the moment you will probably see an error page.

The error you saw was because when we deployed to Heroku, we created a new database and it's empty. We need to run the `migrate` and `createsuperuser` commands, just like we did on PythonAnywhere.  This time, they come via a special command-line on our own computer, `heroku run`:

    $ heroku run python manage.py migrate

    $ heroku run python manage.py createsuperuser

The command prompt will ask you to choose a username and a password again. These will be your login details on your live website's admin page.


Refresh it in your browser, and there you go!  You now know how to deploy to two different hosting platforms.  Pick your favourite :)
