# 파이썬애니웨어(PythonAnywhere) 수동 배포하기

In the main tutorial, we deployed our app using PythonAnywhere's "autoconfigure" script, which did a lot of magic for us.  In this extension we'll take a peek "behind the scenes" and find out what autoconfigure script actually did, by learning how to deploy our code manually to PythonAnywhere.

Why might you be interested in doing this?  Beyond pure curiosity, the steps involved in a manual deployment are more or less the same steps you'd need to go through if you were trying to deploy to a different hosting provider, including running your own server, which is something you might want to do some day.

Running through the procedure manually is also a good idea for DjangoGirls coaches.  If something goes wrong with the autoconfigure script in one of the workshops, knowing how to do it manually could help you get one of the attendees out of a broken state and back to a working deployment.

So, read on!


## Assumptions

* This extension assumes you already have some code on GitHub.  See the begining of the deploy chapter for instructions on that, if you need.
* It also assumes you have a PythonAnywhere account.  If it already has a djangogirls webapp running in it, you'll need to delete that first (or just sign up for a new account!)

## Overview

Deploying to PythonAnywhere, or indeed to any server, involves pretty much the same steps:

* Getting your code onto the server.  We'll use `git clone` and `git pull` for this

* Installing your dependencies on the server.  We'll use a virtualenv for this, just like on your own computer, but we'll use a shortcut recommended by PythonAnywhere called `virtualenvwrapper`

* Configuring your database on the server.  The database on your own computer and the one on the server are separate.  We'll use `manage.py migrate` and `manage.py createsuperuser` for this.

* Configuring your static files on the server.  On your own laptop, `runserver` takes care of static files, but servers like to manage things differently in order to optimise serving static files.  Here we'll use a new command called `collectstatic`, and configure the static files through PythonAnywhere's **Web tab**.

* And actually hooking up our Django app to be served, live, on the public Internet.  We do this on PythonAnywhere's **Web tab**, by configuring something they call a **WSGI file**.


## Pulling our code down on PythonAnywhere

Go to the [PythonAnywhere Dashboard](https://www.pythonanywhere.com), and choose the option to start a "Bash" console – that's the PythonAnywhere version of a command-line, just like the one on your computer.

<img src="images/pythonanywhere_bash_console.png" alt="pointing at Other: Bash in Start a new Console" />

> **Note** PythonAnywhere is based on Linux, so if you're on Windows, the console will look a little different from the one on your computer.

Let's pull down our code from GitHub and onto PythonAnywhere by creating a "clone" of our repo. Type the following into the console on PythonAnywhere:

{% filename %}PythonAnywhere command-line{% endfilename %}
```
$ git clone https://github.com/<your-github-username>/my-first-blog.git <your-pythonanywhere-username>.pythonanywhere.com
```

* Use your actual GitHub username in place of `<your-github-username>`
* And use your actual PythonAnywhere username in place of `<your-pythonanywhere-username>`)

This will pull down a copy of your code onto PythonAnywhere, and put it into a folder named after your (future) website address. Check it out by typing `tree`:

{% filename %}PythonAnywhere command-line{% endfilename %}
```
$ tree
ola.pythonanywhere.com/
├── blog
│   ├── __init__.py
│   ├── admin.py
│   ├── migrations
│   │   ├── 0001_initial.py
│   │   └── __init__.py
│   ├── models.py
│   ├── tests.py
│   └── views.py
├── manage.py
└── mysite
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```


### Creating a virtualenv on PythonAnywhere

Just like you did on your own computer, you need to create a virtualenv on PythonAnywhere. In the Bash console, type:

{% filename %}PythonAnywhere command-line{% endfilename %}
```
$ mkvirtualenv --python=python3.6 <your-pythonanywhere-username>.pythonanywhere.com
Running virtualenv with interpreter /usr/bin/python3.6
[...]
Installing setuptools, pip...done.
```

`mkvirtualenv` comes from a tool called "virtualenvwrapper", which PythonAnywhere recommends.  It's a set of shortcuts built around the normal `virtualenv` command that you've already learned about using on your own computer.

When the command finishes, your virtualenv should be active; we've named it after your future website address, just like we named the project source code folder.  Let's try de-activating and re-activating the virtualenv, just for practice.

Deactivating is `deactivate`, just like on your computer, but to activate we use the virtualenvwrapper shortcut command `workon`, which just needs the name of your virtualenv:

{% filename %}PythonAnywhere command-line{% endfilename %}
```
(ola.pythonanywhere.com) $ deactivate
$  which python
/usr/bin/python
$  workon <your-pythonanywhere-username>.pythonanywhere.com
(ola.pythonanywhere.com) $ which python
/home/ola/.virtualenvs/ola.pythonanywhere.com/bin/python
```

Now let's install Django into our virtualenv on PythonAnywhere

{% filename %}PythonAnywhere command-line{% endfilename %}
```
(ola.pythonanywhere.com) $ pip install 'django<2'
Collecting django
[...]
Successfully installed django-1.11.9
```

> **Note** The `pip install` step can take a couple of minutes.  Patience, patience!  But if it takes more than five minutes, something is wrong.  Ask your coach.

<!--TODO: think about using requirements.txt instead of pip install.-->


### Creating the database on PythonAnywhere

Here's another thing that's different between your own computer and the server: it uses a different database. So the user accounts and posts can be different on the server and on your computer.

Just as we did on your own computer, we repeat the step to initialize the database on the server, with `migrate` and `createsuperuser`:

Go back to your Bash console, make sure your virtualenv is still active, and then run these commands. (If you've closed your console, you can open a new one, and use the `workon` command to activate your virtualenv).

{% filename %}PythonAnywhere command-line{% endfilename %}
```
(ola.pythonanywhere.com) $ python manage.py migrate
Operations to perform:
[...]
  Applying sessions.0001_initial... OK
(ola.pythonanywhere.com) $ python manage.py createsuperuser
```

### Collecting static files

Now we learn a new command, `collecstatic`, whose job it is to collect all the static files from your apps (including apps you've written like *blog*, and built-in Django apps like the *admin*), and put them in one place, so the server can find them:

{% filename %}PythonAnywhere command-line{% endfilename %}
```
(ola.pythonanywhere.com) $ python manage.py collectstatic
You have requested to collect static files at the destination
[...]
Are you sure you want to do this?
[...]
62 static files copied to '/home/ola/ola.pythonanywhere.com/static'.
```


## Publishing our blog as a web app

Now our code is on PythonAnywhere, our virtualenv is ready, and the database is initialized. We're ready to publish it as a web app!

Click back to the PythonAnywhere dashboard by clicking on its logo, and then click on the **Web** tab. Finally, hit **Add a new web app**.

After confirming your domain name, choose **manual configuration** (N.B. – *not* the "Django" option) in the dialog. Next choose **Python 3.6**, and click Next to finish the wizard.

> **Note** Make sure you choose the "Manual configuration" option, not the "Django" one. We're too cool for the default PythonAnywhere Django setup. ;-)


### Setting the virtualenv

You'll be taken to the PythonAnywhere config screen for your webapp, which is where you'll need to go whenever you want to make changes to the app on the server.

<img src="images/pythonanywhere_web_tab_virtualenv.png" alt="screenshot of web tab virtualenv config section with path correctly filled in" />

In the "Virtualenv" section, click the red text that says "Enter the path to a virtualenv", and enter `/home/<your-PythonAnywhere-username>/my-first-blog/myvenv/`. Click the blue box with the checkmark to save the path before moving on.

> **Note** Substitute your own PythonAnywhere username as appropriate. If you make a mistake, PythonAnywhere will show you a little warning.


### Adding the static files mapping

We need to tell PythonAnywhere that static files which live under the URL `/static/` are all located in a file inside your source code folder called static. We do that in the "Static Files" section on the Web tab.

Click the red text that says "Enter URL", and enter `/static/`, and click the blue checkbox to save.  Then click the text that says "Enter path", and enter `/home/<your-pythonanywhere-username>/<your-pythonanywhere-username.pythonanywhere.com/static` (using your own username, as usual):

<img src="images/static_files_screenshot.png" alt="screenshot of web tab static files config section with url and path correctly filled in" />




### Configuring the WSGI file

Django works using the "WSGI protocol", a standard for serving websites using Python, which PythonAnywhere supports. The way we configure PythonAnywhere to recognize our Django blog is by editing a WSGI configuration file.

Click on the "WSGI configuration file" link (in the "Code" section near the top of the page – it'll be named something like `/var/www/<your-PythonAnywhere-username>_pythonanywhere_com_wsgi.py`), and you'll be taken to an editor.

Delete all the contents and replace them with the following:

{% filename %}&lt;your-username&gt;_pythonanywhere_com_wsgi.py{% endfilename %}
```python
import os
import sys

path = '/home/<your-pythonanywhere-username>/<your-pythonanywhere-username>.pythonanywhere.com')
if path not in sys.path:
    sys.path.append(path)

os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'

from django.core.wsgi import get_wsgi_application
application = get_wsgi_application()
```

> **Note** As always, substitute your own PythonAnywhere username in this file.


This file's job is to tell PythonAnywhere where our web app lives and what the Django settings file's name is.

Hit **Save** and then go back to the **Web** tab.

We're all done! Hit the big green **Reload** button and you'll be able to go view your application. You'll find a link to it at the top of the page.


## Debugging tips

If you see an error when you try to visit your site, the first place to look for some debugging info is in your **error log**. You'll find a link to this on the PythonAnywhere [Web tab](https://www.pythonanywhere.com/web_app_setup/). See if there are any error messages in there; the most recent ones are at the bottom. Common problems include:

- Forgetting one of the steps we did in the console: creating the virtualenv, activating it, installing Django into it, migrating the database.

- Making a mistake in the virtualenv path on the Web tab – there will usually be a little red error message on there, if there is a problem.

- Making a mistake in the WSGI configuration file – did you get the path to your my-first-blog folder right?

- Did you pick the same version of Python for your virtualenv as you did for your web app? Both should be 3.6.

There are also some [general debugging tips on the PythonAnywhere wiki](https://www.pythonanywhere.com/wiki/DebuggingImportError).

