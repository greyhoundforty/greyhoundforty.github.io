---
layout: post
title: Configure Terraform for the IBM Cloud
date: 2019-01-31 15:31 -0600
---

## Overview
Today I will be showing you how to configure Terraform for use with the IBM Cloud. 

<!--description-->

## Prerequisites
- [Terraform installed](https://www.terraform.io/intro/getting-started/install.html)
- [IBM Cloud Provider Binary installed](https://github.com/IBM-Cloud/terraform-provider-ibm/releases)
- [IBM Cloud API key](https://cloud.ibm.com/docs/iam/userid_keys.html#userapikey)
- [IBM Cloud IaaS/SoftLayer username and API key](https://cloud.ibm.com/docs/iam/classic_infra_keys.html#classic_keys)

## Configure Terraform  
Create a `~/.terraformrc` file that points to the Terraform binary. For example if you installed the binary to `/usr/local/bin/terraform-provider-ibm` the `~/.terraformrc` would look like this:

```hcl
providers {
    ibm = "/usr/local/bin/terraform-provider-ibm"
}
```

## Configure Plugin to work with Terraform

To provide your credentials as environment variables, you can use the following code in your `provider.tf` file.

```hcl
provider "ibm" {
   bluemix_api_key    = "${var.ibm_bx_api_key}"
   softlayer_username = "${var.ibm_sl_username}"
   softlayer_api_key  = "${var.ibm_sl_api_key}"
}
```

Be sure to also define at minimum the following variables in your `variables.tf` file:

```hcl
variable ibm_bx_api_key {}
variable ibm_sl_username {}
variable ibm_sl_api_key {}
```

For any example that also deploys a PaaS offering you may also need to add:

```
variable ibm_account_guid {}
variable ibm_org_guid {}
variable ibm_space_guid {}
```

If you do not know your IBM Account/Org/Space GUID you can run the following commands to obtain them:

```shell
# Get account guid 
$ ibmcloud iam accounts

# Get Org guid
$ ibmcloud iam orgs --guid

# Get Space guid
$ ibmcloud iam space <SPACE NAME> --guid
```

With those gathered you can now export your environmental variables for use with Terraform. 

```shell
export TF_VAR_ibm_bx_api_key="$VALUE"
export TF_VAR_ibm_sl_username="$VALUE"
export TF_VAR_ibm_sl_api_key="$VALUE"
export TF_VAR_ibm_account_guid="$VALUE"  # If needed
export TF_VAR_ibm_org_guid="$VALUE"  # If needed
export TF_VAR_ibm_space_guid="$VALUE"  # If needed
```

## Testing your configuration
In order to test if we have everything configured properly we can use the following example code to spin up a `transient` instance on the IBM Cloud. The first step is to clone the example code repository:

```
$ git clone https://github.com/greyhoundforty/ibm-cloud-terraform-starter.git
$ cd ibm-cloud-terraform-starter
```

Now we can configure our `credentials.tfvars` file by copying the example version and configuring it with our IBM Cloud information.

```
$ cp credentials.tfvars.tmpl credentials.tfvars
$ nano/vim/editor credentials.tfvars
```

With the credentials configured we can now prepare our system to deploy our VSI:

```
terraform init 
```

The `terraform init` command is used to initialize a working directory containing Terraform configuration files. This is the first command that should be run after writing a new Terraform configuration or cloning an existing one from version control. It is safe to run this command multiple times.

If there were no errors with our configuration we can now run the `plan` command. This command will look at our current environment and create an execution plan for what actions need to be taken for the environment to match the desired state listed in the configuration files. The `-out` flag is used to save this execution plan to a file that can be used when we `apply` our configuration.

```
terraform plan -var-file='./credentials.tfvars' -out test.tfplan
```

You should see a nice terminal print out of what needs to be done. For a net-new environment it would look something like this:

```
Terraform will perform the following actions:

  + ibm_compute_vm_instance.node
      id:                           <computed>
      block_storage_ids.#:          <computed>
      cores:                        <computed>
      datacenter:                   "dal13"
      disks.#:                      <computed>
      domain:                       "example.com"
      file_storage_ids.#:           <computed>
      flavor_key_name:              "B1_1X2X100"
      hostname:                     "${random_id.name.hex}-test"
      hourly_billing:               "true"
      ip_address_id:                <computed>
      ip_address_id_private:        <computed>
      ipv4_address:                 <computed>
      ipv4_address_private:         <computed>
      ipv6_address:                 <computed>
      ipv6_address_id:              <computed>
      ipv6_enabled:                 "false"
      ipv6_static_enabled:          "false"
      local_disk:                   "false"
      memory:                       <computed>
      network_speed:                "1000"
      os_reference_code:            "UBUNTU_18_64"
      private_interface_id:         <computed>
      private_network_only:         "false"
      private_security_group_ids.#: <computed>
      private_subnet:               <computed>
      private_subnet_id:            <computed>
      private_vlan_id:              <computed>
      public_bandwidth_limited:     <computed>
      public_bandwidth_unlimited:   "false"
      public_interface_id:          <computed>
      public_ipv6_subnet:           <computed>
      public_ipv6_subnet_id:        <computed>
      public_security_group_ids.#:  <computed>
      public_subnet:                <computed>
      public_subnet_id:             <computed>
      public_vlan_id:               <computed>
      secondary_ip_addresses.#:     <computed>
      tags.#:                       "1"
      tags.1224436441:              "terraform-test"
      transient:                    "true"
      wait_time_minutes:            "90"

  + random_id.name
      id:                           <computed>
      b64:                          <computed>
      b64_std:                      <computed>
      b64_url:                      <computed>
      byte_length:                  "4"
      dec:                          <computed>
      hex:                          <computed>


Plan: 2 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

This plan was saved to: test.tfplan

To perform exactly these actions, run the following command to apply:
    terraform apply "test.tfplan"
```

To deploy the VSI we run the `apply` command on our existing plan file:

```
$ terraform apply test.tfplan

random_id.name: Creating...
  b64:         "" => "<computed>"
  b64_std:     "" => "<computed>"
  b64_url:     "" => "<computed>"
  byte_length: "" => "4"
  dec:         "" => "<computed>"
  hex:         "" => "<computed>"
random_id.name: Creation complete after 0s (ID: oQDmMw)
ibm_compute_vm_instance.node: Creating...
  block_storage_ids.#:          "" => "<computed>"
  cores:                        "" => "<computed>"
  datacenter:                   "" => "dal13"
  disks.#:                      "" => "<computed>"
  domain:                       "" => "example.com"
  file_storage_ids.#:           "" => "<computed>"
  flavor_key_name:              "" => "B1_1X2X100"
  hostname:                     "" => "a100e633-test"
  hourly_billing:               "" => "true"
  ip_address_id:                "" => "<computed>"
  ip_address_id_private:        "" => "<computed>"
  ipv4_address:                 "" => "<computed>"
  ipv4_address_private:         "" => "<computed>"
  ipv6_address:                 "" => "<computed>"
  ipv6_address_id:              "" => "<computed>"

<output output output>

ibm_compute_vm_instance.node: Still creating... (3m0s elapsed)
ibm_compute_vm_instance.node: Still creating... (3m10s elapsed)
ibm_compute_vm_instance.node: Creation complete after 3m10s (ID: 70547143)

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

## Clean up

To clean up our test deployment we will run the `destroy` command to cancel off our transient VSI.

```
$ terraform destroy -var-file='./credentials.tfvars'
```