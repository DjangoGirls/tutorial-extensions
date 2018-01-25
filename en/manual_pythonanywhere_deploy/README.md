# Deploying to PythonAnywhere manually

In the main tutorial, we deployed our app using PythonAnywhere's "autoconfigure" script, which did a lot of magic for us.  In this extension we'll take a peek "behind the scenes" and find out what autoconfigure script actually did, by learning how to deploy our code manually to PythonAnywhere.

Why might you be interested in doing this?  Beyond pure curiosity, the steps involved in a manual deployment are more or less the same steps you'd need to go through if you were trying to deploy to a different hosting provider, including running your own server, which is something you might want to do some day.

Running through the procedure manually is also a good idea for DjangoGirls coaches.  If something goes wrong with the autoconfigure script in one of the workshops, knowing how to do it manually could help you get one of the attendees out of a broken state and back to a working deployment.

So, read on!


## Assumptions

* This extension assumes you already have some code on GitHub.  See the begining of the deploy chapter for instructions on that, if you need.
* It also assumes you have a PythonAnywhere account.  If it already has a djangogirls webapp running in it, you'll need to delete that first (or just sign up for a new account!)


## Pulling our code down on PythonAnywhere

Go to the [PythonAnywhere Dashboard](https://www.pythonanywhere.com), and choose the option to start a "Bash" console – that's the PythonAnywhere version of a command-line, just like the one on your computer.

<img src="images/pythonanywhere_bash_console.png" alt="pointing at Other: Bash in Start a new Console" />

> **Note** PythonAnywhere is based on Linux, so if you're on Windows, the console will look a little different from the one on your computer.

Let's pull down our code from GitHub and onto PythonAnywhere by creating a "clone" of our repo. Type the following into the console on PythonAnywhere (don't forget to use your GitHub username in place of `<your-github-username>`):

{% filename %}PythonAnywhere command-line{% endfilename %}
```
$ git clone https://github.com/<your-github-username>/my-first-blog.git
```

This will pull down a copy of your code onto PythonAnywhere. Check it out by typing `tree my-first-blog`:

{% filename %}PythonAnywhere command-line{% endfilename %}
```
$ tree my-first-blog
my-first-blog/
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

Just like you did on your own computer, you can create a virtualenv on PythonAnywhere. In the Bash console, type:

{% filename %}PythonAnywhere command-line{% endfilename %}
```
$ cd my-first-blog

$ virtualenv --python=python3.6 myvenv
Running virtualenv with interpreter /usr/bin/python3.6
[...]
Installing setuptools, pip...done.

$ source myvenv/bin/activate

(myvenv) $  pip install django~=1.11.0
Collecting django
[...]
Successfully installed django-1.11.3
```


> **Note** The `pip install` step can take a couple of minutes.  Patience, patience!  But if it takes more than five minutes, something is wrong.  Ask your coach.

<!--TODO: think about using requirements.txt instead of pip install.-->


### Creating the database on PythonAnywhere

Here's another thing that's different between your own computer and the server: it uses a different database. So the user accounts and posts can be different on the server and on your computer.

Just as we did on your own computer, we repeat the step to initialize the database on the server, with `migrate` and `createsuperuser`:

{% filename %}PythonAnywhere command-line{% endfilename %}
```
(mvenv) $ python manage.py migrate
Operations to perform:
[...]
  Applying sessions.0001_initial... OK
(mvenv) $ python manage.py createsuperuser
```


## Publishing our blog as a web app

Now our code is on PythonAnywhere, our virtualenv is ready, and the database is initialized. We're ready to publish it as a web app!

Click back to the PythonAnywhere dashboard by clicking on its logo, and then click on the **Web** tab. Finally, hit **Add a new web app**.

After confirming your domain name, choose **manual configuration** (N.B. – *not* the "Django" option) in the dialog. Next choose **Python 3.6**, and click Next to finish the wizard.

> **Note** Make sure you choose the "Manual configuration" option, not the "Django" one. We're too cool for the default PythonAnywhere Django setup. ;-)


### Setting the virtualenv

You'll be taken to the PythonAnywhere config screen for your webapp, which is where you'll need to go whenever you want to make changes to the app on the server.

<img src="images/pythonanywhere_web_tab_virtualenv.png" />

In the "Virtualenv" section, click the red text that says "Enter the path to a virtualenv", and enter `/home/<your-PythonAnywhere-username>/my-first-blog/myvenv/`. Click the blue box with the checkmark to save the path before moving on.

> **Note** Substitute your own PythonAnywhere username as appropriate. If you make a mistake, PythonAnywhere will show you a little warning.


### Configuring the WSGI file

Django works using the "WSGI protocol", a standard for serving websites using Python, which PythonAnywhere supports. The way we configure PythonAnywhere to recognize our Django blog is by editing a WSGI configuration file.

Click on the "WSGI configuration file" link (in the "Code" section near the top of the page – it'll be named something like `/var/www/<your-PythonAnywhere-username>_pythonanywhere_com_wsgi.py`), and you'll be taken to an editor.

Delete all the contents and replace them with the following:

{% filename %}&lt;your-username&gt;_pythonanywhere_com_wsgi.py{% endfilename %}
```python
import os
import sys

path = os.path.expanduser('~/my-first-blog')
if path not in sys.path:
    sys.path.append(path)

os.environ['DJANGO_SETTINGS_MODULE'] = 'mysite.settings'

from django.core.wsgi import get_wsgi_application
from django.contrib.staticfiles.handlers import StaticFilesHandler
application = StaticFilesHandler(get_wsgi_application())
```

This file's job is to tell PythonAnywhere where our web app lives and what the Django settings file's name is.

The `StaticFilesHandler` is for dealing with our CSS. This is taken care of automatically for you during local development by the `runserver` command. We'll find out a bit more about static files later in the tutorial, when we edit the CSS for our site.

Hit **Save** and then go back to the **Web** tab.

We're all done! Hit the big green **Reload** button and you'll be able to go view your application. You'll find a link to it at the top of the page.


## Debugging tips

If you see an error when you try to visit your site, the first place to look for some debugging info is in your **error log**. You'll find a link to this on the PythonAnywhere [Web tab](https://www.pythonanywhere.com/web_app_setup/). See if there are any error messages in there; the most recent ones are at the bottom. Common problems include:

- Forgetting one of the steps we did in the console: creating the virtualenv, activating it, installing Django into it, migrating the database.

- Making a mistake in the virtualenv path on the Web tab – there will usually be a little red error message on there, if there is a problem.

- Making a mistake in the WSGI configuration file – did you get the path to your my-first-blog folder right?

- Did you pick the same version of Python for your virtualenv as you did for your web app? Both should be 3.6.

There are also some [general debugging tips on the PythonAnywhere wiki](https://www.pythonanywhere.com/wiki/DebuggingImportError).

