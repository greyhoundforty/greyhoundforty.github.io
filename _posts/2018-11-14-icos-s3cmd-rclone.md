---
layout: post
title:  IBM Cloud Object storage with s3cmd or rclone
category: ibmcloud
---

# Overview

Today I will show you how to interact with IBM Cloud Object storage using some command line utilities. While the IBM Cloud Object storage supports IAM and API key based access, most third party utilities require you to use the `secret key/access key` method of authentication so I will also be showing you how to generate `HMAC` credentials for use with `rclone` and `s3cmd`.

<!--description-->

## Generating HMAC Credentials via the IBM Cloud CLI

If you have the IBM Cloud CLI installed you can generate the needed `secret key/access key` combo by using the `service-key-create` command. First I will find the instance on my account. I tend to forget things if I try to be cute and clever with names so in this case I named my Cloud Object Storage instance `icos-rt`. With the instance name we can now generate some service credentials. To have the new credentials include `secret key/access key` authentication we have to pass the JSON parameters `{"HMAC":true}` to the `service-key-create` command. 

{% highlight shell %}
$ ibmcloud resource service-key-create icos-rt-creds Administrator --instance-name icos-rt -p '{"HMAC":true}'
Creating service key of service instance icos-rt under account IBM as rtiffany@us.ibm.com...
OK
Service key crn:v1:bluemix:public:cloud-object-storage:global:a/xxxxxxx:xxxxxxxx:resource-key:xxxxxx was created.

Name:          icos-rt-creds
(output output output)
{% endhighlight %}


## Generating HMAC Credentials via the IBM Cloud Portal

To generate HMAC credentials via the GUI, simply go to the IBM Cloud Object storage account and click on `Service Credentials`. On the subsequent page click `New Credential` and under `Add Inline Configuration Parameters` enter `{"HMAC":true}`.

> ![](https://dsc.cloud/quickshare/Screenshot_2018-11-14-11.48.04_tEP2cy.png)

> ![](https://dsc.cloud/quickshare/Screenshot_2018-11-15-15.19.51_qPDey0.png)


### s3cmd 
The first tool I will highlight is [`s3cmd`](https://github.com/s3tools/s3cmd){:target="_blank"}

> S3cmd (s3cmd) is a free command line tool and client for uploading, retrieving and managing data in Amazon S3 and other cloud storage service providers that use the S3 protocol, such as Google Cloud Storage or DreamHost DreamObjects. It is best suited for power users who are familiar with command line programs. It is also ideal for batch scripts and automated backup to S3, triggered from cron, etc. 
>
> -- from the Official s3cmd repository

#### Installing s3cmd

You can grab the most up to date version of s3cmd from the [s3cmd releases](https://github.com/s3tools/s3cmd/releases) page. Unpack the binary and put it in `$PATH`. Once the binary is in place you can run through the initial configuration by issuing the command `s3cmd --configure`. You will be prompted for your Secret Key, Access Key, the COS [endpoint](https://console.bluemix.net/docs/services/cloud-object-storage/basics/endpoints.html#select-regions-and-endpoints) you would like to use as well as some additional options around encryption. With the configuration complete you can now test that everything is configured by listing the buckets on your account. 

{% highlight shell %}

$ s3cmd ls
2018-06-25 17:11  s3://cloudcontainerqstest
2018-07-02 03:04  s3://homenextcloud
2018-08-14 17:33  s3://icospublicaccesstestbucket
2018-10-02 19:46  s3://image-template-exports-rt
2018-10-13 01:05  s3://isoimporttemplates
2018-09-10 18:44  s3://johnnykaratessuperawesomebucket
2018-09-13 19:51  s3://k8s-pvc-backup-bucket
2018-08-09 18:55  s3://k8spvcbuckettest
2018-07-27 19:20  s3://lingering-feather-73
2018-05-04 18:08  s3://objectivefstest
2018-04-23 19:27  s3://publicaccesstestbucket
2018-04-27 01:14  s3://remote-tf-statefiles
2018-10-26 20:54  s3://rtvsibackupbucket
2018-04-25 20:44  s3://testingasperahighspeedthing
{% endhighlight %}

[Full s3cmd documentation][s3cmddocs]

### rclone 

The second tool I want to highlight is [`rclone`][rclone]{:target="_blank"}

> Rclone ("rsync for cloud storage") is a command line program to sync files and directories to and from different cloud storage providers.

To get started with rclone head over to [rclone downloads](https://rclone.org/downloads/) and grab the approriate package for your operating system. Once you have `rclone` installed you can head over to this handy [guide][Configure rclone to work with IBM Cloud Object Storage] for a run through of how to configure rclone to work with IBM Cloud Object storage. Your configuration file should end up looking something like this:

{% highlight shell %}
[IBMCOS]
type = s3
provider = IBMCOS
env_auth = false
access_key_id = xxxxxxxxxxxx
secret_access_key = xxxxxxxxxxxx
endpoint = s3-api.us-geo.objectstorage.softlayer.net
location_constraint = us-standard
acl = public-read
{% endhighlight %}

With the tool configured you can test it out by listing the buckets on your account:

{% highlight shell %}
$ rclone lsd IBMCOS:
          -1 2018-06-25 12:11:20        -1 cloudcontainerqstest
          -1 2018-07-01 22:04:59        -1 homenextcloud
          -1 2018-08-14 12:33:31        -1 icospublicaccesstestbucket
          -1 2018-10-02 14:46:53        -1 image-template-exports-rt
          -1 2018-10-12 20:05:54        -1 isoimporttemplates
          -1 2018-09-10 13:44:27        -1 johnnykaratessuperawesomebucket
          -1 2018-09-13 14:51:31        -1 k8s-pvc-backup-bucket
          -1 2018-08-09 13:55:02        -1 k8spvcbuckettest
          -1 2018-07-27 14:20:35        -1 lingering-feather-73
          -1 2018-05-04 13:08:07        -1 objectivefstest
          -1 2018-04-23 14:27:12        -1 publicaccesstestbucket
          -1 2018-04-26 20:14:33        -1 remote-tf-statefiles
          -1 2018-10-26 15:54:46        -1 rtvsibackupbucket
          -1 2018-04-25 15:44:55        -1 testingasperahighspeedthing

$ rclone lsd IBMCOS:johnnykaratessuperawesomebucket
     0 2018-11-14 09:37:11        -1 Sync
     0 2018-11-14 09:37:11        -1 terraform_logs

{% endhighlight %}

[Full rclone documentation][full rclone docs]

[s3cmddocs]: https://s3tools.org/usage
[rclone]: https://rclone.org/
[Configure rclone to work with IBM Cloud Object Storage]: https://rclone.org/s3/#ibm-cos-s3
[full rclone docs]: https://rclone.org/docs/