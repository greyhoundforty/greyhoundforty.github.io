---
layout: post
title:  Getting started with the IBM Cloud CLI
category: ibmcloud 
---

## Overview
Today I will be showing you how to get up and running with deploying a simple web application backed by a hosted IBM Cloud service. I will be utilizing the IBM Cloud CLI and will show a few different deployment options. 

<!--description-->

## Target audience
This guide is really aimed at folks that have dabbled with command line and maybe even done a bit of light programming work but are unfamiliar with deploying applications to the IBM Cloud. 

## Prerequisites
- An IBM Cloud Account
- Access to Linux or macOS terminal

### Installing CLI

You have a few options for how you would like to install the IBM Cloud CLI:

1. Run the one-line installer. This will also install some IBM Cloud Plugins as well as tools to interact with Kubernetes and Docker. See here for the full list of installed components. Piping to bash is controversial, as it prevents you from reading code that is about to run on your system. Therefore, we provide these alternative installation methods:
2. Download and install the binary version. If you're comfortable unziping files and understand how to move binaries in to your PATH then this is the option for you.
3. Use [bxshell](https://github.com/l2fprod/bxshell) which is run as a local Docker container that has the CLI and tools baked in. Pretty handy for kicking the tires.

### Installing plugins

If you choose option 1 in the previous step you will also need to install a few plugins to follow along with this tutorial.

{% highlight shell %}
$ ibmcloud plugin install dev -r 'IBM Cloud'
$ ibmcloud plugin install container-service/kubernetes-service -r 'IBM Cloud' 
$ ibmcloud plugin install container-registry -r 'IBM Cloud'
{% endhighlight %}

With the CLI and plugins installed we can move on to logging in to the CLI and getting our environment configured.

### CLI Authentication

It is not considered best practice to log in to the CLI using your IBMid username and password, so we will invoke the `--sso` option which will open a browser window for account log in. Once you have logged in to the account via the browser you will be redirected to a page that has a one-time passcode to use for authentication.

{% highlight shell %}
$ ibmcloud login --sso
API endpoint: https://api.ng.bluemix.net

Get One Time Code from https://iam-id-2.eu-gb.bluemix.net/identity/passcode to proceed.
Open the URL in the default browser? [Y/n]>Y
{% endhighlight %}

![IBM Cloud Login page](https://dsc.cloud/quickshare/GUI-Loginv2.png)

![One Time Passcode](https://dsc.cloud/quickshare/onetimepass.png)

Once we've got the code we hop back to the terminal and complete the log in process.

{% highlight shell %}
One Time Code > xxxxxx
Authenticating...
OK

Select an account:
1. Mouserat Account (xxxxxxxxxxxxxxxxxxxxxxx)
2. IBM (xxxxxxxxxxxxxxxxxxxxxxx) <-> 78003
Enter a number> 2
Targeted account IBM (xxxxxxxxxxxxxxxxxxxxxxx) <-> 78003

(output output output)
{% endhighlight %}

### Generating an API Key

Since we've already touched on not wanting to use IBMid and password for authentication with the CLI, and generating a one-time passcode is not always practical (headless server for instance), we can create an API key that will be used for authentication instead.

{% highlight shell %}
$ ibmcloud iam api-key-create <KEY-NAME> -d "Key Description"
{% endhighlight %}

You will need to copy and paste the key somewhere safe as it will not be displayed again.

Set environmental variable

{% highlight shell %}
export IBMCLOUD_API_KEY='<key copied from previous step>'
{% endhighlight %}

Setting Resource group, Region, and Space


You can target specific pieces of your IBM Cloud environment during the login process using the IBM Cloud CLI. For instance on my account I want to target the CDE TEAM Organization and under that the Resource Group CDE and the Space coolkids in the US-South region:

{% highlight shell %}
$ ibmcloud login -a api.ng.bluemix.net -o "CDE TEAM" -g "CDE" -s coolkids

Targeted account IBM (xxxxxxxxxxxxxxxxxxxxxxx) <-> 78003
Targeted resource group CDE
Targeted Cloud Foundry (https://api.ng.bluemix.net)
Targeted org CDE TEAM
Targeted space coolkids
(output output output)
{% endhighlight %}

## Creating a starter application using the IBM Cloud CLI Dev plugin

Now that we're logged in and ready to get up and running we'll invoke the `ibmcloud dev create` command to walk us through the process of creating our Web app:

<a href="https://asciinema.org/a/208069" target="_blank"><img src="https://asciinema.org/a/208069.svg" /></a>

### Updating our starter application

As you can see in the video above, the `dev` tool created our application code, bound an IBM Cloud service (in this case Cloudant), and downloaded the code to our local directory. With the code now on our machine we can build our application and test it locally using the build and run subcommands:

{% highlight shell %}
$ cd ~/rtgotestapp
~/rtgotestapp $ ibmcloud dev build
Getting service credentials for the application.
OK
Validating Docker image name
OK
Checking if Docker container rtgotestapp-go-tools is running
OK
Checking Docker image history to see if image already exists
OK
Creating image rtgotestapp-go-tools based on Dockerfile-tools ...
Image will have user ryan with id 501 added

Executing docker image build --file Dockerfile-tools --tag rtgotestapp-go-tools --rm --pull --build-arg bx_dev_userid=501 --build-arg bx_dev_user=ryan .

OK
Creating a container named 'rtgotestapp-go-tools' from that image...
OK
Starting the 'rtgotestapp-go-tools' container...
OK
OK
Stopping the 'rtgotestapp-go-tools' container...
OK

~/rtgotestapp $ ibmcloud dev run
The run-cmd option was not specified
Stopping the 'rtgotestapp-go-run' container...
The 'rtgotestapp-go-run' container was not found
Validating Docker image name
Binding IP and ports for Docker image.
OK
Checking if Docker container rtgotestapp-go-run is running
OK
Checking Docker image history to see if image already exists
OK
Creating image rtgotestapp-go-run based on Dockerfile ...

Executing docker image build --file Dockerfile --tag rtgotestapp-go-run --rm --pull .

OK
Creating a container named 'rtgotestapp-go-run' from that image...
OK
Starting the 'rtgotestapp-go-run' container...
OK
{% endhighlight %}


From another terminal session we can curl our local application and should see the console return logging information:

{% highlight shell %}
$ curl -I localhost:8080 

Logs for the rtgotestapp-go-run container:
{"level":"info","msg":"Starting rtgotestapp on port :8080","time":"2018-10-23T17:25:24Z"}
[GIN] 2018/10/23 - 17:26:25 | 200 |    1.689152ms |      172.17.0.1 |  GET     /
[GIN] 2018/10/23 - 17:26:31 | 200 |    1.041034ms |      172.17.0.1 |  HEAD     /
[GIN] 2018/10/23 - 17:26:38 | 200 |     188.675µs |      172.17.0.1 |  HEAD     /
[GIN] 2018/10/23 - 17:26:44 | 200 |     391.497µs |      172.17.0.1 |  GET     /
{% endhighlight %}

### Tie in to logging and metrics

### Deployment options (devops,source control)