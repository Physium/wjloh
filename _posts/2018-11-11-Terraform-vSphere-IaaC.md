---
title: "Terraform with vSphere"
categories: 
  - IaaC
tags:
  - Terraform
  - vSphere
toc: true
---

Over the weekend instead of accomplishing my entire TO-DO list, I kinda procastinated and went totally of the track instead. I've have always wanted to check out what this whole **"Infrastructure as a Code"** thing is all about and I have heard quite abit about Terraform being one of the solution that can help achieve this. 

So with Terraform, you can make calls to vSphere to provision VMs (Virtual Machines) without the need of accessing the vSphere/vCenter Client interface. All this is done via writing your own configuration file with Terraform's own configuration language, HCL.

## Getting Started

Considering that I'm on macOS, getting started is as easy as:
```bash
brew install Terraform
```

If you are on Windows, I believe Windows now has a package manager called **[Chocolatey](https://chocolatey.org/)** where you can simply run:

```bash
choco install Terraform
```

## The Basics

At the very high level, a typical Terraform configuration consist of the following:

* Providers
* Resources
* Terraform Lifecycle - ``init``,``plan`` ,``apply`` ,``destory``
* States

## Spinning up a VM on vSphere with Terraform

I've decided to start off with one of the tempate that they have provided **[here](https://www.terraform.io/docs/providers/vsphere/index.html#argument-reference)**. This template pretty much covers the basic usage of spinning up a VM in vSphere with Terraform. But there seems to be some missing components that prevents the code from succesfully executing. I will be addressing that along the way.

What I'll do with the example is that I'll break them up into different components and explain what is needed to be done in each code block.

### Setting up the Provider

First lets start of with the **Provider**. The provider component acts as the first step in terms of initalizing the entire Terraform setup. Given that Terraform works with a large variety of infrastrucutre providers be it your public cloud or on-prem offerings, this is where we tell Terraform what provider(s) are we working with for this particular setup. Essentially, the provider is responsible for the understanding API interactions and exposing resources for us.

What you need to do here is input your vCenter/vSphere Client login credentials and the hostname of the vCenter/vSphere server.

```ruby
provider "vsphere" {
  user           = "wjloh@vsphere.local"
  password       = "JustaRandomPassword"
  vsphere_server = "vcenter01.winterfell.lan"

  # If you have a self-signed cert
  allow_unverified_ssl = true
}
```

Once that is done, we can proceed to carry out ``terraform init`` which you will then see the following which confirms that initailization has successfully taken place.


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

### Assigning Data Sources 

>*Data sources* allow data to be fetched or computed for use elsewhere in Terraform configuration. Use of data sources allows a Terraform configuration to build on information defined outside of Terraform, or defined by another separate Terraform configuration.

When we move to the resource section of setting up the type of VM that we want, the *data sources* essentially makes it alot easier for us to input the required id.

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

The code segment here basically declares the type of resource you want to create, in this case its a VM. There are several configurations with regards to the VM that you can set. What is shown below is the minimum required to get a VM up and runninng. For more information on what can be configured, refer to the full documentation of the vsphere_virtural_machine **[here](https://www.terraform.io/docs/providers/vsphere/r/virtual_machine.html)**.

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
### Running Terraform Plan

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

### Running Terraform Apply

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

  Enter a value: 
```

### Getting it to succesfully execute

So in example that was provided, upon executing ``terraform apply`` what I realise that it actually succesfully spins up the VM in my vCenter. However, in the terminal it's stuck in this never ending loop of *'Still Creating...'* . What I found out was that it seems like it was trying to make a connection to the VM and since there wasnt any OS or IP Address in the VM, it kept waiting for the VM to get one till it timeout.

After a little bit of googling around, I found out what I should have done was to base my resource off a VM template. So I created a simple CentOS VM and converted it to a temaplate.

Next, I added the following to the configurations:
* Data source to the template
* Clone parameters in the VM resource

```ruby
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

  #Included a clone parameter in the resource
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
Now, executing ``terraform apply`` will end off with a success instead. 


