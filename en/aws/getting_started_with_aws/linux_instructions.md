## Installing AWS CLI on Linux
1. Download the installation file from your browser using [this link](https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip) 
or this link if you are using [Linux ARM](https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip).
2. In a command line window, navigate to the folder with your download and unzip the file in the current directory using the command below.

{% filename %} command line {% endfilename %}
```
$ unzip awscliv2.zip
```
3. Run the command below to install aws-cli.

{% filename %} command line {% endfilename %}
```
$ sudo ./aws/install
```
4. Confirm the installation with the command below.

{% filename %} command line {% endfilename %}
```
$ aws --version
```
