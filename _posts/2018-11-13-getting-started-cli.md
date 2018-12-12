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
- Docker installed. You can find the appropriate instructions for your Operating System [here][docker install].

### Installing CLI

You have a few options for how you would like to install the IBM Cloud CLI:

1. Run the one-line installer. This will also install some IBM Cloud Plugins as well as tools to interact with Kubernetes and Docker. See [here][ibmcli oneliner]{:target="_blank"} for the full list of installed components. Piping to bash is controversial, as it prevents you from reading code that is about to run on your system. Therefore, we provide these alternative installation methods:
2. Download and install the [binary version][binary version]{:target="_blank"}. If you're comfortable unziping files and understand how to move binaries in to your PATH then this is the option for you.
3. Use [bxshell][bxshell]{:target="_blank"} which is run as a local Docker container that has the CLI and tools baked in. Pretty handy for kicking the tires.

### Installing plugins

To install the plugins needed for this tutorial we will use the `ibmcloud plugin` subcommand. You can see all the plugins available by running the command `ibmcloud plugin repo-plugins -r 'IBM Cloud'`.  If you choose option 1 in the previous step you should have most of the plugins needed to follow along with this tutorial but it won't hurt to run these commands to ensure they are installed and up to date.

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

Targeted account Ryan Tiffany's Account (xxxxxxxxxxxxxxx) <-> xxxxxxx

Targeted resource group default


API endpoint:      https://api.ng.bluemix.net
Region:            us-south
User:              ryan@example.com
Account:           Ryan Tiffany's Account (xxxxxxxxxxxxxxx) <-> xxxxxxx
Resource group:    default
CF API endpoint:
Org:
Space:

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

You can target specific pieces of your IBM Cloud environment during the login process using the IBM Cloud CLI. For instance on my account I want to target the `Tinylab` Organization and under that the Resource Group `default` and the Space `dev` in the US-South region:

{% highlight shell %}
$ ibmcloud target --cf -o TheTinylab -s dev

Targeted Cloud Foundry (https://api.ng.bluemix.net)
Targeted org TheTinylab
Targeted space Dev
(output output output)
{% endhighlight %}

## Creating a starter application using the IBM Cloud CLI Dev plugin
Now that we're logged in and ready to get up and running we'll invoke the `ibmcloud dev create` command to walk us through the process of creating our Web app. In this guide we won't be setting up a DevOps toolchain deployment so select `3` when prompted about deployment options. 

<a href="https://asciinema.org/a/216612" target="_blank"><img src="https://asciinema.org/a/216612.svg" /></a>

### Building and testing our starter application
As you can see in the video above, the `dev` tool created our application code, bound an IBM Cloud service (in this case Cloudant), and downloaded the code to our local directory. If you created an application via the GUI you can invoke the command `ibmcloud dev code <name of app>` to download the code in to your current directory. 

With the code now on our machine we can build our application and test it locally using the build and run subcommands:

##### Building
{% highlight shell %}
$ cd ~/rtgoappv3
$ ibmcloud dev build

Getting service credentials for the application.
OK
Validating Docker image name
OK
Checking if Docker container rtgoappv3-go-tools is running
OK
Checking Docker image history to see if image already exists
OK
Creating image rtgoappv3-go-tools based on Dockerfile-tools ...
Image will have user ryan with id 1001 added

Executing docker image build --file Dockerfile-tools --tag rtgoappv3-go-tools --rm --pull --build-arg bx_dev_userid=1001 --build-arg bx_dev_user=ryan .

OK
Creating a container named 'rtgoappv3-go-tools' from that image...
OK
Starting the 'rtgoappv3-go-tools' container...
OK
OK
Stopping the 'rtgoappv3-go-tools' container...
OK
{% endhighlight %}

#### Running Locally
{% highlight shell %}
$ ibmcloud dev run 

The run-cmd option was not specified
Stopping the 'rtgoappv3-go-run' container...
The 'rtgoappv3-go-run' container was not found
Validating Docker image name
Binding IP and ports for Docker image.
OK
Checking if Docker container rtgoappv3-go-run is running
OK
Checking Docker image history to see if image already exists
OK
Creating image rtgoappv3-go-run based on Dockerfile ...

Executing docker image build --file Dockerfile --tag rtgoappv3-go-run --rm --pull .

OK
Creating a container named 'rtgoappv3-go-run' from that image...
Waiting for Docker image to build ⠚ OK
Starting the 'rtgoappv3-go-run' container...
OK
{% endhighlight %}

From another terminal session we can curl our local application and should see the console return logging information:

{% highlight shell %}
$ curl -I localhost:8080 

Logs for the rtgoappv3-go-run container:
{"level":"info","msg":"Starting rtgoappv3 on port :8080","time":"2018-12-12T17:07:40Z"}
[GIN] 2018/12/12 - 17:08:13 | 200 |     1.19781ms |      172.17.0.1 |  HEAD     /
[GIN] 2018/12/12 - 17:08:20 | 200 |     210.292µs |      172.17.0.1 |  HEAD     /
[GIN] 2018/12/12 - 17:08:20 | 200 |     204.505µs |      172.17.0.1 |  HEAD     /
{% endhighlight %}

If your `dev build` or `dev run` commands fail, check out some common errors and how to [troubleshoot them](https://cloud.ibm.com/docs/cli/ts_createapps.html#troubleshoot).

### Cloud Foundry Deployment 
With our simple application tested and built we can now move on to deployment. By default when you invoke `ibmcloud dev deploy` the application will be deployed using Cloud Foundry. 

{% highlight shell %}
$ ibmcloud dev deploy 
The hostname for this application will be: rtgoappv3
? Press [Return] to accept this, or enter a new value now>

Provisioning user-defined Cloud Foundry service(s)... ⠂
Executing ibmcloud cf cups Cloudant-rt -p {<REDACTED>}

Deploying to Cloud Foundry...

Executing ibmcloud cf push

Invoking 'cf push'...

Pushing from manifest to org TheTinylab / space Dev as ryan@example.com...
Using manifest file /home/ryan/rtgoappv3/manifest.yml
Using manifest file /home/ryan/rtgoappv3/manifest.yml

Creating app rtgoappv3 in org TheTinylab / space Dev as ryan@example.com...
OK

Creating route rtgoappv3.mybluemix.net...
OK

Binding rtgoappv3.mybluemix.net to rtgoappv3...
OK

App started
OK

App rtgoappv3 was started using this command `./bin/rtgoappv3`

Showing health and status for app rtgoappv3 in org TheTinylab / space Dev as ryan@cdetest.info...
OK

requested state: started
instances: 1/1
usage: 256M x 1 instances
urls: rtgoappv3.mybluemix.net
last uploaded: Wed Dec 12 17:11:30 UTC 2018
stack: cflinuxfs2
buildpack: https://github.com/cloudfoundry/go-buildpack.git#v1.8.25

     state     since                    cpu    memory      disk      details
#0   running   2018-12-12 05:13:17 PM   0.0%   0 of 256M   0 of 1G

OK
Your application is hosted at http://rtgoappv3.mybluemix.net/
{% endhighlight %}

### Kubernetes Deployment
To deploy your application on Kubernetes, you must either specify the deploy-target as `container` in the cli-config.yml or use the parameter `-t container` when invoking `ibmcloud dev deploy`. If you specify the deploy target in the cli-config.yml file you will also need to add some additional options in order to ensure you are targeting the correct cluster:

{% highlight shell %}
    deploy-image-target: "registry.<IBM Cloud Region>.bluemix.net/<Container Registry Namespace>/<App-Name>"

    ibm-cluster: "mycluster"
{% endhighlight %}

The Kubernetes deployment uses [Helm Charts][helm charts]. The `deploy` command will install the server side component `tiller` by default, but for better security I would recommend creating a Service Account on your Kubernetes cluster for Helm.

{% highlight shell %}
$ kubectl create -f https://raw.githubusercontent.com/IBM-Cloud/kube-samples/master/rbac/serviceaccount-tiller.yaml 
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller created

$  helm init --service-account tiller 
$HELM_HOME has been configured at /home/ryan/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
{% endhighlight %}


{% highlight shell %}
$ ibmcloud dev deploy -t container

You are currently logged in. Your application will be deployed to the IBM
Cloud.

The IBM cluster name for the deployment of this application will be:
iks-rt

? Press [Return] to accept this, or enter a new value now>
Log in to the IBM Container Registry
OK
Configuring with cluster 'iks-rt'
OK

The container deployment image name for the deployment of this application
will be: (The image will be tagged by Docker with this base name and appended
with a version)

registry.ng.bluemix.net/shinyhappynamespace/rtgoappv3
? Press [Return] to accept this, or enter a new value now>

Executing helm get values rtgoappv3
Release "rtgoappv3" does not exist. Installing it now.

NAME:   rtgoappv3
LAST DEPLOYED: Wed Dec 12 18:41:27 2018
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                                  READY  STATUS             RESTARTS  AGE
rtgoappv3-deployment-c85cd7695-zqkjg  0/1    ContainerCreating  0         0s

==> v1/Service

NAME                           AGE
rtgoappv3-application-service  0s

==> v1beta1/Deployment
rtgoappv3-deployment  0s
OK

Pods:
NAME                                   READY   STATUS              RESTARTS   AGE
rtgoappv3-deployment-c85cd7695-zqkjg   0/1     ContainerCreating   0          1s

Deployments:
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
rtgoappv3-deployment   1         1         1            0           1s

Services:
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes                      ClusterIP   172.21.0.1       <none>        443/TCP          1h
rtgoappv3-application-service   NodePort    172.21.209.254   <none>        8080:31617/TCP   1s

Nodes:
NAME            STATUS   ROLES    AGE   VERSION
10.209.37.208   Ready    <none>   1h    v1.11.5+IKS
10.209.37.213   Ready    <none>   1h    v1.11.5+IKS

Untag the Run image from the local Docker registry

Executing docker rmi registry.ng.bluemix.net/shinyhappynamespace/rtgoappv3:0.0.1

OK
Your application is hosted at http://169.61.30.194:31617/
{% endhighlight %}

![Simple go app deployed to Kubernetes](https://dsc.cloud/quickshare/Screenshot_2018-12-12-12.42.20_yYpaj1.png)

### Logging with IBM Cloud Log Analysis
If you don't already have an instance of the [IBM Cloud Log Analysis][ibmLogAnalysis] service you can use the CLI to create one.

{% highlight shell %}
$ ibmcloud service create ibmLogAnalysis standard logging-rt
Invoking 'cf create-service ibmLogAnalysis standard logging-rt'...

Creating service instance logging-rt in org TheTinylab / space Dev as ryan@example.com...
OK
{% endhighlight %}

With the logging service instance now deployed we can now view the logs for our Cloud Foundry application using the CLI or the Kibana dashboard.

{% highlight x %}
$ ibmcloud cf logs rtgoappv3 

Invoking 'cf logs rtgoappv3'...
Retrieving logs for app rtgoappv3 in org TheTinylab / space Dev as ryan@example.com...

   2018-12-12T18:50:10.20+0000 [RTR/22] OUT rtgoappv3.mybluemix.net - [2018-12-12T18:50:10.182+0000] "GET / HTTP/1.1" 200 0 56379 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:63.0) Gecko/20100101 Firefox/63.0" "10.184.41.4:57601" "169.46.115.120:61318" x_forwarded_for:"76.224.107.167, 10.184.41.4" x_forwarded_proto:"http" vcap_request_id:"2286ebf5-41b8-4d0a-7884-0a0704f97937" response_time:0.025794918 app_id:"f8ae455b-c6e4-4569-a97d-0d442da37548" app_index:"0" x_global_transaction_id:"3490874319" true_client_ip:"-" x_b3_traceid:"5a8ddd44d94eefec" x_b3_spanid:"5a8ddd44d94eefec" x_b3_parentspanid:"-"
   2018-12-12T18:50:10.20+0000 [RTR/22] OUT
   2018-12-12T18:49:53.77+0000 [APP/PROC/WEB/0] OUT [GIN] 2018/12/12 - 18:49:53 | 200 |    6.342807ms |  76.224.107.167 |  GET     /
   2018-12-12T18:49:53.90+0000 [RTR/26] OUT rtgoappv3.mybluemix.net - [2018-12-12T18:49:53.885+0000] "GET /favicon.ico HTTP/1.1" 200 0 32935 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:63.0) Gecko/20100101 Firefox/63.0" "10.184.41.4:59952" "169.46.115.120:61318" x_forwarded_for:"76.224.107.167, 10.184.41.4" x_forwarded_proto:"http" vcap_request_id:"b4816d9f-1e43-42a9-54af-96c1fdc31319" response_time:0.018047205 app_id:"f8ae455b-c6e4-4569-a97d-0d442da37548" app_index:"0" x_global_transaction_id:"2048179073" true_client_ip:"-" x_b3_traceid:"3d0dda24beeb7988" x_b3_spanid:"3d0dda24beeb7988" x_b3_parentspanid:"-"
{% endhighlight %}

![](https://dsc.cloud/quickshare/kibana_dashboard.png)


[bxshell]: https://github.com/l2fprod/bxshell
[ibmLogAnalysis]: https://cloud.ibm.com/docs/services/CloudLogAnalysis/log_analysis_ov.html#log_analysis_ov
[ibmcli oneliner]: https://cloud.ibm.com/docs/cli/index.html#overview
[binary version]: https://cloud.ibm.com/docs/cli/reference/ibmcloud/all_versions.html#ibm-cloud-cli-releases
[docker install]: https://docs.docker.com/install/
