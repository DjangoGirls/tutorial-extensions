# Deploying a Django app using AWS Elastic Beanstalk

AWS Elastic Beanstalk is a managed service that can handle our deployment for you. All you need is to push our code and it
will handle the provisioning of all the compute and networking resources for you which is awesome! 

To deploy to AWS using Elastic Beanstalk, we first need to install `awsebcli` which we will do below.


# Install AWS Elastic Beanstalk Command Line Tool (CLI)
To deploy to AWS using Elastic Beanstalk, we need to install the Elastic Beanstalk command line tool, `awsebcli`. 
We already have the other Python prerequisites for Elastic Beanstalk already installed, that is `Python >= 3.7`, `pip` 
and `virtualenv`.

It is recommended to install `awsebcli` using the [EB CLI setup scripts](https://github.com/aws/aws-elastic-beanstalk-cli-setup) 
which can be installed by following the following steps:

{% filename %}command-line{% endfilename %}
```
anna $ git clone https://github.com/aws/aws-elastic-beanstalk-cli-setup.git
Cloning into 'aws-elastic-beanstalk-cli-setup'...
remote: Enumerating objects: 325, done.
remote: Counting objects: 100% (30/30), done.
remote: Compressing objects: 100% (21/21), done.
remote: Total 325 (delta 9), reused 22 (delta 6), pack-reused 295
Receiving objects: 100% (325/325), 533.74 KiB | 3.38 MiB/s, done.
Resolving deltas: 100% (172/172), done.
(myvenv) djangogirls $ 
```

To run the installer on Mac OSX/Linux, run the following command:
{% filename %}command-line{% endfilename %}
```
$ python ./aws-elastic-beanstalk-cli-setup/scripts/ebcli_installer.py
```

If on Windows run the following command instead:
{% filename %}command-line{% endfilename %}
```
> python .\aws-elastic-beanstalk-cli-setup\scripts\ebcli_installer.py
```

> Note: If the [EB CLI setup scripts](https://github.com/aws/aws-elastic-beanstalk-cli-setup) setup does not work for 
your platform, you can install `awsebcli` manually. 

On Mac OSX, you can use `Homebrew`. First make sure you have the latest version of Homebrew by running the following command:

{% filename %} command line {% endfilename %}
```
$ brew update
```

{% filename %} command line {% endfilename %}
```
anna $ brew install awsebcli
```

On Linux/Windows, you can use the following command to install `awsebcli`:

{% filename %} command line {% endfilename %}
```
$ pip install awsebcli --upgrade --user
```

To verify the installation was successful, you can run the following command:

{% filename %} command line {% endfilename %}
```
$ eb --version
```

and you should get the version of `awsebcli` you have installed. 

Once `awsebcli` is installed successful, we are ready to prepare our project for deployment to AWS.

## Configuring your Django project for AWS Elastic Beanstalk

To deploy to AWS using AWS Elastic Beanstalk, your project should have the following:
- a `requirements.txt` file which it uses to install the packages required to run your website in the EC2 instance.
- a configuration file, `django.config` which specifies the WSGI file that Elastic Beanstalk will use to start your project.

Let's generate the two files and prepare our project for deployment.

### Creating a requirements.txt file

To create a `requirements.txt` for your project, first navigate to your project folder and activate your virtual environment. 

To navigate to your project folder, type the following command:

```
$ cd djangogirls
```

On Linux/Mac OSX, run the following command:

```
djangogirls $ source myvenv/bin/activate
```

On Windows, run the following command:

```
> myvenv/Scripts/activate
```

Next, run the following command to create a `requirements.txt` file in the root of your project:

```
(myvenv) djangogirls $ pip freeze > requirements.txt
```

If your check your project folder, it should now have a `requirements.txt` file. 
The folder structure for your project should now look like below:

```
~/djangogirls
  |-- blog
  |-- mysite
  |   |-- __init__.py
  |   |-- asgi.py
  |   |-- settings.py
  |   |-- urls.py
  |   `-- wsgi.py
  |-- myvenv
  |-- .gitignore
  |-- db.sqlite3
  |-- manage.py
  |-- requirements.txt
```

>>> Note, we have only expanded `mysite` folder only because we are interested in its contents and did not expand 
`myvenv` and `blog` folders as their contents are irrelevant for this tutorial.

Now let's create the config file for Elastic Beanstalk.

### Creating a configuration file for Elastic Beanstalk
To create a configuration file for ELastic Beanstalk, first create a new directory called `.ebextensions` as shown below.

```
(myvenv) djangogirls $ mkdir .ebextensions
```

Next add a configuration file in the `.ebextensions` folder using your text editor called `django.config` and add the 
following content in it:

```
option_settings:
  aws:elasticbeanstalk:container:python:
    WSGIPath: mysite.wsgi:application
```

The setting `WSGIPath` specifies the location of the project's WSGI file which Elastic Beanstalk will to launch your
app.

## Deploying to AWS with Elastic Beanstalk

You are now ready to deploy to AWS using Elastic Beanstalk. Your project directory should now look like below:

```
~/djangogirls
  |-- .ebextensions
  |   `-- django.config
  |-- blog
  |-- mysite
  |   |-- __init__.py
  |   |-- asgi.py
  |   |-- settings.py
  |   |-- urls.py
  |   `-- wsgi.py
  |-- myvenv
  |-- .gitignore
  |-- db.sqlite3
  |-- manage.py
  |-- requirements.txt
```  

To create your application environment and deploy your configured application with Elastic Beanstalk, run the following 
command in command line:

```
djangogirls $ eb init -p python-3.8 my-first-blog
Application my-first-blog has been created
```

The command creates an application named `my-first-blog` and configures your local repository to create environments with latest
Python 3.8 platform version. You can specify another Python version like 3.9, 3.10, 3.11, 3.12 etc we used 3.8 because this is what
our main tutorial is based on.

Next, run `eb init` again to initialize SSH so you can be able to connect to the EC2 instance running your application. 
You will be presented many prompts with guidelines on how you should respond. 
In the example below, I used defaults but you can name things differently as you please, just be sure to remember the names.

```
djangogirls $ eb init
Do you wish to continue with CodeCommit? (Y/n): Y

Enter Repository Name
(default is "origin"): 
Successfully created repository: origin

Enter Branch Name
***** Must have at least one commit to create a new branch with CodeCommit *****
(default is "main"): 
Successfully created branch: main
Do you want to set up SSH for your instances?
(Y/n): Y

Type a keypair name.
(Default is aws-eb): 
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
```

> Note: To change the settings later you can run `eb init -i` command.

To create an environment and deploy your application to it run the following command:

```
djangogirls $ eb create my-first-blog
```

This will create a load-balanced Elastic Beanstalk environment named `my-first-blog`. 
When an environment is load-balanced, it means that requests will be shared between various instances/web servers so that no 
one server is overloaded with requests. 
Elastic Beanstalk will scale up and down to meet demand by spinning up or down more instances as required so that application 
performance is not degraded during peak hours. 

The process of creating an environment takes about 5 minutes, during which Elastic Beanstalk creates the resources needed to run your application. 
During the process, EBI CLI will relay output informational messages to your terminal.

```
djangogirls $ eb status
Environment details for: django-env
Application name: my-blog
Region: us-west-2
Deployed Version: app-6852-230925_133132320894
Environment ID: e-wwp53kf7g6
Platform: arn:aws:elasticbeanstalk:us-west-2::platform/Python 3.8 running on 64bit Amazon Linux 2/3.5.6
Tier: WebServer-Standard-1.0
CNAME: django-env.eba-nvwz6wvx.us-west-2.elasticbeanstalk.com
Updated: 2023-09-25 11:36:38.424000+00:00
Status: Ready
Health: Red
```

Copy the `CNAME` and add it to the `ALLOWED_HOSTS` setting in `mysite/settings.py`.

Run the following command to deploy the changes to your website:

```
djangogirls $ eb deploy
```

And run this command to open your website:

```
djangogirls $ eb open
```

