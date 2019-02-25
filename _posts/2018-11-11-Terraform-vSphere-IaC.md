---
title: "Provisioning VMs on vSphere with Terraform"
categories: 
  - SDDC
tags:
  - Terraform
  - vSphere
  - Infrastructure as a Code
  - Automation
  - IaC
toc: true
---

Over the weekend instead of accomplishing my entire TO-DO list, I kinda procrastinated and went totally off track. I've have always wanted to check out what this whole **"Infrastructure as Code"** hype is all about and I have heard quite the buzz about Terraform being one of the solution that can help achieve this. 

## What is Infrastructure as Code?

First lets take a look at the definition,

> Infrastructure as Code (IaC) is a method to provision and manage IT infrastructure through the use of source code, rather than through standard operating procedures and manual processes.

Well, from what I understand its literally what the name says it is. The idea of provisioning and managing your infrastructure via code sounds interesting. But why do that? If we think about it, we are essentially treating our infrastructure like software. This code can help you configure and deploy your infrastructure components quickly and consistently. The ability to automate infrastructure deployment in a repeatable, consistent manner provides many benefits! I will probably leave the specifics of the benefits and why IaC is the future for another day.

### Terra... what?

So with the introduction of Terraform, it basically simplifies the overall process of writing code to provision and manage your infrastructure. As Terraform comes with its own configuration engine & language known as [HCL](https://github.com/hashicorp/hcl), it helps to mask away a lot of the back-end complexities and simplifies the infrastructure configurations process.

For more information on what Terraform is, checkout the official documentation.
* [What is Terraform?](https://www.terraform.io/intro/index.html)
* [Use Cases](https://www.terraform.io/intro/use-cases.html)

## Overview

I have segment this tutorial into 3 main sections:

1. [Installing Terraform](#1-terraform-installation)
2. [Getting Started with Terraform Configuration](#2-getting-started-with-terraform-configurations)
3. [Executing Terraform Configurations](#3-executing-terraform-configurations)

This way its a lot easier for you to navigate around and jump right into your area of choice. 

At the point where I wrote this post, the software that I used to run the examples are of the following:
* Terraform v0.11.10
* Terraform vSphere Provider v1.9.0 
* vSphere 6.7U1

**Now, let's get started!**

## 1. Terraform Installation
Considering that I'm on macOS, getting Terraform up and running is as easy as:
```bash
brew install Terraform
```

If you are on Windows, I believe Windows now has a package manager called **[Chocolatey](https://chocolatey.org/)** where you can simply run:
```bash
choco install Terraform
```

If you do not wish to use a package manager for your installation, please have a look at the offical documentation **[here](https://www.terraform.io/intro/getting-started/install.html)** on how to set up Terraform on your machine.

## 2. Getting Started with Terraform Configurations

At the very high level, a typical Terraform configuration consist of the following:

* Providers
* Data Sources
* Resources
* Variables

I've decided to start off with their example configuration template that was provided **[here](https://www.terraform.io/docs/providers/vsphere/index.html#argument-reference)**. This template pretty much covers the basic usage of spinning up a VM in vSphere with Terraform. But there seems to be some missing components that prevents the code from successfully executing. I will be addressing that along the way.

What I'll do next is that I will breakdown the example into different components and explain what is needed to be done in each configuration block.

### Setting up the Provider

First lets start of with the **Provider**. The provider component acts as the first step in terms of initializing the entire Terraform setup. Given that Terraform works with a large variety of infrastructure providers be it your public cloud or on-premise offerings, this is where we tell Terraform what [provider(s)](https://www.terraform.io/docs/providers/index.html) we are working with for this particular setup. Essentially, the provider is responsible for the understanding API interactions and exposes resources for us.

In this example, we will be using [vSphere as the provider](https://www.terraform.io/docs/providers/vsphere/index.html). What you need to do here is input your vCenter login credentials and the hostname of the vCenter server. The example also showcases the usage of variables but lets skip that for now.

```ruby
provider "vsphere" {
  user           = "wjloh@vsphere.local"
  password       = "JustaRandomPassword"
  vsphere_server = "vcenter01.winterfell.lan"

  # If you have a self-signed cert
  allow_unverified_ssl = true
}
```

### Assigning Data Sources 

>*Data sources* allows data to be fetched or computed for use elsewhere in Terraform configuration. Use of data sources allows a Terraform configuration to build on information defined outside of Terraform, or defined by another separate Terraform configuration.

Essentially, this is how we fetch vSphere related information from our environment and define it at as a data source object which will then be used as part of the VM resource configuration.

What we do here is change the **name** parameter to the name of our resources that is set within our environment.
```ruby
data "vsphere_datacenter" "dc" {
  name = "Winterfell"
}

data "vsphere_datastore" "datastore" {
  name          = "vsanDatastore"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}

data "vsphere_resource_pool" "pool" {
  name          = "vSAN Cluster/Resources"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}

data "vsphere_network" "network" {
  name          = "VM-Network-DVPG"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}
```

### Creating Resources

>*Resources* are a component of your infrastructure. It might be some low level component such as a physical server, virtual machine, or container. Or it can be a higher level component such as an email provider, DNS record, or database provider.

This segment here declares the type of resource you want to create, in this case its a VM. There are several configurations with regards to the VM that you can set. What is shown below is the minimum required to get a VM up and running. For more information on what can be configured, refer to the full documentation of the ``vsphere_virtural_machine`` **[here](https://www.terraform.io/docs/providers/vsphere/r/virtual_machine.html)**.

```ruby
resource "vsphere_virtual_machine" "vm" {
  name             = "terraform-test"
  resource_pool_id = "${data.vsphere_resource_pool.pool.id}"
  datastore_id     = "${data.vsphere_datastore.datastore.id}"

  num_cpus = 2
  memory   = 1024
  guest_id = "other3xLinux64Guest"

  network_interface {
    network_id = "${data.vsphere_network.network.id}"
  }

  disk {
    label = "disk0"
    size  = 50
  }
}
```

### Troubleshooting

Upon executing the configuration, I realize that it actually successfully spins up the VM in my vCenter. However, in the terminal it's stuck in this never ending loop of *'Still Creating...'* which was very puzzling. What I found out was that it was trying to make a connection to the VM and since there wasn't any OS or IP Address in the VM, it kept waiting for the VM to be assigned a connection till it timeout. For more information on this, you can read about it **[here](https://www.terraform.io/docs/providers/vsphere/r/virtual_machine.html#customization-and-network-waiters)**.

After a little bit of Googling around, I found out there are 2 ways fix this.

1. I could simply add the following to the resource configuration to tell Terraform to not wait for any IP Address configuration.
```ruby
 wait_for_guest_net_timeout = 0
```

2. I could clone a VM using a VM template and customize it with a valid routable IP address. With that, I created a simple CentOS VM and converted it to a template.

Next, I added the following to the configurations:
* A data source to retrieve the VM template
* A clone attribute in the VM resource

```ruby
#Data source for VM template
data "vsphere_virtual_machine" "template" {
  name = "CentOSVM"
  datacenter_id = "${data.vsphere_datacenter.dc.id}"
}

resource "vsphere_virtual_machine" "vm" {
  name             = "terraform-test"
  resource_pool_id = "${data.vsphere_resource_pool.pool.id}"
  datastore_id     = "${data.vsphere_datastore.datastore.id}"

  num_cpus = 1
  memory   = 1024
  guest_id = "centos7_64Guest"

  network_interface {
    network_id = "${data.vsphere_network.network.id}"
  }

  disk {
    label = "disk0"
    size  = 50
  }

  #Included a clone attribute in the resource
  clone {
    template_uuid = "${data.vsphere_virtual_machine.template.id}"

    customize {
      linux_options{
        host_name = "wjloh"
        domain = "winterfell.lan"
      }
      network_interface {
        ipv4_address = "10.206.1.112"
        ipv4_netmask = "24"
      }

      ipv4_gateway = "10.206.1.1"
      dns_suffix_list = ["winterfell.lan"]
      dns_server_list = ["10.206.1.10"]
    }
  }
}
```
Now, executing it will end off with a success instead.

## 3. Executing Terraform Configurations

Now, I'll talk about how to execute the configuration files.

The lifecycle of running and managing Terraform configurations comes in 4 parts.
* init
* plan
* apply
* destroy

### Initializing Terraform

We first start off by running ``terraform init`` . Terraform will proceed to initialize the provider(s) which you have indicated in your configuration file. 

The following is an example output which confirms that the initialization has successfully taken place.

```bash
$ terraform init

Initializing provider plugins...
- Checking for available provider plugins on https://releases.hashicorp.com...
- Downloading plugin for provider "vsphere" (1.9.0)...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.vsphere: version = "~> 1.9"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

### Terraform Plan

Next, you can run ``terraform plan`` which gives you a good overview on what are the final configurations that will be executed. Here is a good time to review your configurations and check if you would like to make any changes.

```bash
$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

data.vsphere_datacenter.dc: Refreshing state...
data.vsphere_datastore.datastore: Refreshing state...
data.vsphere_network.network: Refreshing state...
data.vsphere_resource_pool.pool: Refreshing state...

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + vsphere_virtual_machine.vm
      id:                                        <computed>
      boot_retry_delay:                          "10000"
      change_version:                            <computed>
      cpu_limit:                                 "-1"
      cpu_share_count:                           <computed>
      cpu_share_level:                           "normal"
      datastore_id:                              "datastore-11"
      default_ip_address:                        <computed>
      disk.#:                                    "1"
      disk.0.attach:                             "false"
      disk.0.datastore_id:                       "<computed>"
      disk.0.device_address:                     <computed>
      disk.0.disk_mode:                          "persistent"
      disk.0.disk_sharing:                       "sharingNone"
      disk.0.eagerly_scrub:                      "false"
      disk.0.io_limit:                           "-1"
      disk.0.io_reservation:                     "0"
      disk.0.io_share_count:                     "0"
      disk.0.io_share_level:                     "normal"
      disk.0.keep_on_remove:                     "false"
      disk.0.key:                                "0"
      disk.0.label:                              "disk0"
      disk.0.path:                               <computed>
      disk.0.size:                               "20"
      disk.0.thin_provisioned:                   "true"
      disk.0.unit_number:                        "0"
      disk.0.uuid:                               <computed>
      disk.0.write_through:                      "false"
      ept_rvi_mode:                              "automatic"
      firmware:                                  "bios"
      force_power_off:                           "true"
      guest_id:                                  "other3xLinux64Guest"
      guest_ip_addresses.#:                      <computed>
      host_system_id:                            <computed>
      hv_mode:                                   "hvAuto"
      imported:                                  <computed>
      latency_sensitivity:                       "normal"
      memory:                                    "1024"
      memory_limit:                              "-1"
      memory_share_count:                        <computed>
      memory_share_level:                        "normal"
      migrate_wait_timeout:                      "30"
      moid:                                      <computed>
      name:                                      "terraform-test"
      network_interface.#:                       "1"
      network_interface.0.adapter_type:          "vmxnet3"
      network_interface.0.bandwidth_limit:       "-1"
      network_interface.0.bandwidth_reservation: "0"
      network_interface.0.bandwidth_share_count: <computed>
      network_interface.0.bandwidth_share_level: "normal"
      network_interface.0.device_address:        <computed>
      network_interface.0.key:                   <computed>
      network_interface.0.mac_address:           <computed>
      network_interface.0.network_id:            "dvportgroup-150"
      num_cores_per_socket:                      "1"
      num_cpus:                                  "2"
      reboot_required:                           <computed>
      resource_pool_id:                          "resgroup-8"
      run_tools_scripts_after_power_on:          "true"
      run_tools_scripts_after_resume:            "true"
      run_tools_scripts_before_guest_shutdown:   "true"
      run_tools_scripts_before_guest_standby:    "true"
      scsi_bus_sharing:                          "noSharing"
      scsi_controller_count:                     "1"
      scsi_type:                                 "pvscsi"
      shutdown_wait_timeout:                     "3"
      swap_placement_policy:                     "inherit"
      uuid:                                      <computed>
      vapp_transport.#:                          <computed>
      vmware_tools_status:                       <computed>
      vmx_path:                                  <computed>
      wait_for_guest_net_routable:               "true"
      wait_for_guest_net_timeout:                "5"


Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

### Terraform Apply

Once you have confirmed on the configurations as shown in ``terraform plan``, the next step would be to apply the configurations and spin up the resource!

```bash
$ terraform apply
data.vsphere_datacenter.dc: Refreshing state...
data.vsphere_datastore.datastore: Refreshing state...
data.vsphere_resource_pool.pool: Refreshing state...
data.vsphere_network.network: Refreshing state...

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  + vsphere_virtual_machine.vm
      id:                                        <computed>
      boot_retry_delay:                          "10000"
      change_version:                            <computed>
      cpu_limit:                                 "-1"
      cpu_share_count:                           <computed>
      cpu_share_level:                           "normal"
      datastore_id:                              "datastore-11"
      default_ip_address:                        <computed>
      disk.#:                                    "1"
      disk.0.attach:                             "false"
      disk.0.datastore_id:                       "<computed>"
      disk.0.device_address:                     <computed>
      disk.0.disk_mode:                          "persistent"
      disk.0.disk_sharing:                       "sharingNone"
      disk.0.eagerly_scrub:                      "false"
      disk.0.io_limit:                           "-1"
      disk.0.io_reservation:                     "0"
      disk.0.io_share_count:                     "0"
      disk.0.io_share_level:                     "normal"
      disk.0.keep_on_remove:                     "false"
      disk.0.key:                                "0"
      disk.0.label:                              "disk0"
      disk.0.path:                               <computed>
      disk.0.size:                               "20"
      disk.0.thin_provisioned:                   "true"
      disk.0.unit_number:                        "0"
      disk.0.uuid:                               <computed>
      disk.0.write_through:                      "false"
      ept_rvi_mode:                              "automatic"
      firmware:                                  "bios"
      force_power_off:                           "true"
      guest_id:                                  "other3xLinux64Guest"
      guest_ip_addresses.#:                      <computed>
      host_system_id:                            <computed>
      hv_mode:                                   "hvAuto"
      imported:                                  <computed>
      latency_sensitivity:                       "normal"
      memory:                                    "1024"
      memory_limit:                              "-1"
      memory_share_count:                        <computed>
      memory_share_level:                        "normal"
      migrate_wait_timeout:                      "30"
      moid:                                      <computed>
      name:                                      "terraform-test"
      network_interface.#:                       "1"
      network_interface.0.adapter_type:          "vmxnet3"
      network_interface.0.bandwidth_limit:       "-1"
      network_interface.0.bandwidth_reservation: "0"
      network_interface.0.bandwidth_share_count: <computed>
      network_interface.0.bandwidth_share_level: "normal"
      network_interface.0.device_address:        <computed>
      network_interface.0.key:                   <computed>
      network_interface.0.mac_address:           <computed>
      network_interface.0.network_id:            "dvportgroup-150"
      num_cores_per_socket:                      "1"
      num_cpus:                                  "2"
      reboot_required:                           <computed>
      resource_pool_id:                          "resgroup-8"
      run_tools_scripts_after_power_on:          "true"
      run_tools_scripts_after_resume:            "true"
      run_tools_scripts_before_guest_shutdown:   "true"
      run_tools_scripts_before_guest_standby:    "true"
      scsi_bus_sharing:                          "noSharing"
      scsi_controller_count:                     "1"
      scsi_type:                                 "pvscsi"
      shutdown_wait_timeout:                     "3"
      swap_placement_policy:                     "inherit"
      uuid:                                      <computed>
      vapp_transport.#:                          <computed>
      vmware_tools_status:                       <computed>
      vmx_path:                                  <computed>
      wait_for_guest_net_routable:               "true"
      wait_for_guest_net_timeout:                "5"


Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

vsphere_virtual_machine.vm: Creating...
  ...
  # a repeat of the configs shown above
  ...
vsphere_virtual_machine.vm: Still creating... (10s elapsed)
...
vsphere_virtual_machine.vm: Creation complete after 2m11s (ID: 422870ff-7786-f80c-44de-8b4743018a0d)

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

Upon completion, head over to your vCenter and you'll see your VM up and running with the configurations that you have assigned!

### Terraform Destroy

Now that you have created a VM, what happens if you wanna remove it? Since the VM is created by Terraform it would be best if the VM is remove via Terraform as well instead of directly deleting it off from the vCenter. This way we can ensure that the **[state](https://www.terraform.io/docs/state/index.html)** files are up to date.

```bash
$ terraform destroy
data.vsphere_datacenter.dc: Refreshing state...
data.vsphere_network.network: Refreshing state...
data.vsphere_resource_pool.pool: Refreshing state...
data.vsphere_datastore.datastore: Refreshing state...
data.vsphere_virtual_machine.template: Refreshing state...
vsphere_virtual_machine.vm: Refreshing state... (ID: 422870ff-7786-f80c-44de-8b4743018a0d)

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  - vsphere_virtual_machine.vm


Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

vsphere_virtual_machine.vm: Destroying... (ID: 422870ff-7786-f80c-44de-8b4743018a0d)
vsphere_virtual_machine.vm: Still destroying... (ID: 422870ff-7786-f80c-44de-8b4743018a0d, 10s elapsed)
vsphere_virtual_machine.vm: Still destroying... (ID: 422870ff-7786-f80c-44de-8b4743018a0d, 20s elapsed)
vsphere_virtual_machine.vm: Destruction complete after 22s

Destroy complete! Resources: 1 destroyed.
```

There you go! You have successfully deleted the VM.

## Final Words

I have compiled the configurations that I wrote above into a gist for easier reference. 

<script src="https://gist.github.com/Physium/83323a8dadc51b3a6d4cbb7ab816dc5c.js"></script>

You just have tweak the `name` variables according to your environment variables.

I'm definitely looking to explore further into the capabilities of Terraform. There seems to be a lot that you do with this and what I have shown is barely just the tip of the iceberg. With the integration of configurations management tools (Chef, Puppet, Packer, etc) I see this as a very powerful automation tool where you get to design and blueprint the entire end to end process of your application in a code like manner.

## References
If you would like to read more about what I just did you can check the following official guides/documentation from Terraform:
* [Getting Started](https://www.terraform.io/intro/getting-started/install.html)
* [Terraform's vSphere Documentation](https://www.terraform.io/docs/providers/vsphere/index.html)
* [Using Infrastructure as Code to Automate VMware Deployments](https://www.hashicorp.com/blog/using-infrastructure-as-code-to-automate-vmware-deployments)
* [A (Re)-Introduction to the Terraform vSphere Provider](https://www.hashicorp.com/blog/a-re-introduction-to-the-terraform-vsphere-provider)

Here are some of the blogs that I reference from while learning about Terraform:
* [Deploying a VM w/ Terraform in ESXi without vCenter](https://elatov.github.io/2018/07/use-terraform-to-deploy-a-vm-in-esxi/)
* [Deploying vSphere VM with Terraform by Emil Wypych](https://emilwypych.com/2017/02/26/deploying-vsphere-vm-with-terraform/)
* [Terraform with vSphere by vGemba](https://www.vgemba.net/vmware/terraform/Terraform-Part-1/)