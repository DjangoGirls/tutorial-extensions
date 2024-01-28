## Installing AWS CLI on Mac OSX
1. Open a terminal window and type the following command to install Homebrew if you Homebrew in not already installed 
on your machine.

{% filename %} command line {% endfilename %}
```
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
2. Type the command below to install AWS CLI using Homebrew.

{% filename %} command line {% endfilename %}
```
$ brew install awscli
```
3. Confirm the installation by running the command below.

{% filename %} command line {% endfilename %}
```
$ aws --version
```
