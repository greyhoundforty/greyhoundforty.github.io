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

1. Run the one-line installer. This will also install some IBM Cloud Plugins as well as tools to interact with Kubernetes and Docker. See [here][ibmcli oneliner] for the full list of installed components. Piping to bash is controversial, as it prevents you from reading code that is about to run on your system. Therefore, we provide these alternative installation methods:
2. Download and install the [binary version][binary version]. If you're comfortable unziping files and understand how to move binaries in to your PATH then this is the option for you.
3. Use [bxshell][bxshell]{:target="_blank"} which is run as a local Docker container that has the CLI and tools baked in. Pretty handy for kicking the tires.

### Installing plugins

If you choose option 1 in the previous step you will also need to install a few plugins to follow along with this tutorial.

{% highlight shell %}
$ ibmcloud plugin install dev -r 'IBM Cloud'
$ ibmcloud plugin install container-service/kubernetes-service -r 'IBM Cloud' 
$ ibmcloud plugin install container-registry -r 'IBM Cloud'
$ ibmcloud plugin install logging-cli -r 'IBM Cloud'
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

As you can see in the video above, the `dev` tool created our application code, bound an IBM Cloud service (in this case Cloudant), and downloaded the code to our local directory. If you created an application via the GUI you can invoke the command `ibmcloud dev code <name of app>` to download the code in to your current directory. 

With the code now on our machine we can build our application and test it locally using the build and run subcommands:

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

### Cloud Foundry Deployment 
With our simple application tested and built we can now move on to deployment. By default when you invoke `ibmcloud dev deploy` the application will be deployed using Cloud Foundry. 

{% highlight shell %}
$ ibmcloud dev deploy

The hostname for this application will be: rtgotestapp
? Press [Return] to accept this, or enter a new value now>
Deploying to Cloud Foundry...

Executing ibmcloud cf push

Invoking 'cf push'...

Pushing from manifest to org CDE TEAM / space coolkids...
Using manifest file /Users/ryan/rtgotestapp/manifest.yml
Using manifest file /Users/ryan/rtgotestapp/manifest.yml

Creating app rtgotestapp in org CDE TEAM / space coolkids...
OK

Creating route rtgotestapp.mybluemix.net...
OK

Binding rtgotestapp.mybluemix.net to rtgotestapp...
OK

Uploading rtgotestapp...
Uploading app files from: /Users/ryan/rtgotestapp
Uploading 20.6K, 35 files
Done uploading
OK

(yada yada yada)

0 of 1 instances running, 1 starting
1 of 1 instances running

App started
OK

App rtgotestingapp was started using this command `./bin/rtgotestapp`
requested state: started
instances: 1/1
usage: 256M x 1 instances
urls: rtgotestapp.mybluemix.net
last uploaded: Tue Nov 27 15:50:50 UTC 2018
stack: cflinuxfs2
buildpack: https://github.com/cloudfoundry/go-buildpack.git#v1.8.25

     state     since                    cpu    memory      disk      details
#0   running   2018-11-27 09:52:49 AM   0.0%   0 of 256M   0 of 1G

OK
Your application is hosted at http://rtgotestapp.mybluemix.net/
{% endhighlight %}

### Kubernetes Deployment

To deploy your application on Kubernetes, you must either specify the deploy-target as `container` in the cli-config.yml or use the parameter `-t container` when invoking `ibmcloud dev deploy`. If you specify the deploy target in the cli-config.yml file you will also need to add some additional options in order to ensure you are targeting the correct cluster:

{% highlight shell %}
    deploy-image-target: "registry.<IBM Cloud Region>.bluemix.net/<Container Registry Namespace>/<App-Name>"

    ibm-cluster: "mycluster"
{% endhighlight %}

{% highlight shell %}
$ ibmcloud dev deploy -t container
You are currently logged in. Your application will be deployed to the IBM
Cloud.

The IBM cluster name for the deployment of this application will be:
rtk8s

? Press [Return] to accept this, or enter a new value now>
Log in to the IBM Container Registry
OK
Configuring with cluster 'rtk8s'
OK
? Select a namespace for this deployment
1. thundercougarfalconbird
2. rtiffany
Enter a number> 2

Executing helm get values rtgotestapp

Inspecting service bindings for cluster 'rtk8s'.
OK
Tag the Run image to the Docker registry as
registry.ng.bluemix.net/rtiffany/rtgotestapp:0.0.1

Executing docker tag rtgotestapp-go-run registry.ng.bluemix.net/rtiffany/rtgotestapp:0.0.1

Unable to tag the image rtgotestapp-go-run, will now attempt to build and tag
it

Executing docker build -f Dockerfile -t registry.ng.bluemix.net/rtiffany/rtgotestapp:0.0.1 .

(yada yada yada)

Release "rtgotestapp" does not exist. Installing it now.
NAME:   rtgotestapp
LAST DEPLOYED: Tue Nov 27 15:33:31 2018
NAMESPACE: default
STATUS: DEPLOYED

Executing docker rmi registry.ng.bluemix.net/rtiffany/rtgotestapp:0.0.1

OK
Your application is hosted at http://169.62.210.230:31774/
{% endhighlight %}

### Logging

If you don't already have an instance of the [IBM Cloud Log Analysis][ibmLogAnalysis] service you can use the CLI to create one. 

{% highlight shell %}
$ ibmcloud service create ibmLogAnalysis standard logging-rt
Invoking 'cf create-service ibmLogAnalysis standard logging-rt'...

Creating service instance logging-rt in org CDE TEAM / space coolkids...
OK
{% endhighlight %}

With the logging service instance now deployed we can now view the logs for our Cloud Foundry application using the CLI or the Kibana dashboard.

{% highlight x %}
$ ibmcloud cf logs rtgotestingapp
Invoking 'cf logs rtgotestingapp'...

Retrieving logs for app rtgotestingapp 

   2018-11-28T20:57:27.04+0000 [RTR/18] OUT rtgotestapp.mybluemix.net - [2018-11-28T20:57:27.025+0000] "GET / HTTP/1.1" 200 0 42 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:63.0) Gecko/20100101 Firefox/63.0" "10.142.73.5:26126" "169.47.199.70:61398" x_forwarded_for:"76.224.107.171, 10.142.73.5" x_forwarded_proto:"http" vcap_request_id:"b95a09b2-341a-476a-5b34-f18c5a2e8d85" response_time:0.018924148 app_id:"98c95cd2-12e4-4bf1-b5a8-8ea49160ae0c" app_index:"0" x_global_transaction_id:"216301923" true_client_ip:"-" x_b3_traceid:"253df64a06a76190" x_b3_spanid:"253df64a06a76190" x_b3_parentspanid:"-"
   2018-11-28T20:57:27.04+0000 [RTR/18] OUT
{% endhighlight %}

![](https://dsc.cloud/quickshare/kibana_dashboard.png)

[bxshell]: https://github.com/l2fprod/bxshell
[ibmLogAnalysis]: https://console.bluemix.net/docs/services/CloudLogAnalysis/log_analysis_ov.html#log_analysis_ov
[ibmcli oneliner]: https://console.bluemix.net/docs/cli/index.html#overview
[binary version]: https://console.bluemix.net/docs/cli/reference/ibmcloud/all_versions.html#ibm-cloud-cli-releases