---
title: "Up & Running with VMware Cloud Foundation (VCF)"
categories: 
  - SDDC
tags:
  - VMware Cloud Foundation
  - VCF
toc: true
---

Its been awhile since I last posted. After much procrastination, I decided to set aside some time during this short December break of mine to write about this topic. 

Recently, I was given an opportunity to help up with the set up of VCF (VMware Cloud Foundation). There has been much talks about how VCF becomes the foundation of SDDC and how it to work across the different cloud offerings creating a seamless hybrid approach. This sits in well with the recent collaboration announcements of VMware and AWS where VCF will be used as the foundation for public cloud offerings in particular [VMware on AWS](https://cloud.vmware.com/vmc-aws) and [AWS outpost](https://aws.amazon.com/outposts/). I was truly excited as I could never try it on my homelab due to hardware and resource constraints.


## What is VCF?

![VCF Overview](/assets/images/vcf/vcf-overview.png "VCF Overview"){: .full}

> Cloud Foundation provides a complete set of software-defined services for compute, storage, network and security, along with cloud management capabilities. The end result is simple, secure and agile cloud
infrastructure that can be can deployed on premises and consumed as a service from public cloud.

Essentially, VCF is an integrated software stack that uses SDDC Manager which is an automated life-cycle management tool to bundle compute (vSphere), storage(vSAN) and network visualization(NSX) into a single platform that can be deployed on premises as a private cloud or run as a service within a public cloud. It fundamentally tries to solve the traditional issues of data centers silos by merging the 3 components together making it easy not only for day one but for day two as well.

Do check out the [product page](https://www.vmware.com/sg/products/cloud-foundation.html) for more info!

## Deploying SDDC w/ VCF

For a start, I'd like to drop a disclaimer on what I will be sharing. A lot of what is being shared below is based on my own personal experience and I may have miss out or not include everything there is when it comes to deploying VCF. Content within this blog post is done with VCF 3.0.x and from what I understand VCF 3.5.x has very recently been GA'ed.

As such, I think it's important to check out the official VCF [documentation](https://docs.vmware.com/en/VMware-Cloud-Foundation/index.html) based on the type of version you are deploying as it may varies. I personally think that the VCF documentation is well documented as it covers all the way from the planning and preparation to the life-cycle management and day 2 operations. 

In this blog, I'll be focusing mainly on the SDDC bring up process with VCF which consist of the following:
1. [Planning & Preparation](#planning--preparationn)
2. [Architecture & Deployment](#architecture--deployment)

## Planning & Preparation

The entire bring up process with VCF is actually pretty straightforward. In order for that to happen, it's important that we get the necessary preparation work done in order for the Cloud Foundation Builder VM to work its magic. I personally wont go into the details of that as there's an entire documentation on the planning and preparation required for the setup to take place. I would strongly suggest using that as a check list to ensure everything is in place before going ahead with the deployment.

Documentation mentioned in this section can be found [here](https://docs.vmware.com/en/VMware-Cloud-Foundation/3.0.1/com.vmware.vcf.planprep.doc_301/GUID-BFF8C3AE-6C42-4133-AACA-00BE0C02B722.html)!

## Architecture & Deployment

### Architecture
There are 2 main architecture designs for for VCF
* [Standard Architecture Model](https://docs.vmware.com/en/VMware-Cloud-Foundation/3.5/com.vmware.vcf.ovdeploy.doc_35/GUID-6C75E7C7-AC9A-41A4-A5D8-AC85BAD4FC1F.html)
* [Consolidated Architecture Model](https://docs.vmware.com/en/VMware-Cloud-Foundation/3.5/com.vmware.vcf.ovdeploy.doc_35/GUID-61453C12-3BB8-4C2A-A895-A1A805931BB2.html)

I believe that the main decision criteria choosing between standard or consolidated would be the number of hosts you have. It's recommended that if you have lesser then 6 hosts is best you go with a consolidated model. But not to worry, as you can shift to a standard model anytime when you see the need to scale.

### Deployment

Now... here's where the magic happens. The deployment process of Cloud Foundation consist of the following:
1. [Deploy Cloud Foundation Builder VM](#1-deploy-cloud-foundation-builder-vm)
2. [Fill up configuration excel sheet](#2-fill-up-the-configuration-excel-sheet)
3. [Validate Excel Sheet Configurations](#3-validate-excel-sheet-configurations)
4. [Sit Back and Grab a Coffee](#4-sit-back-and-relax)

#### 1. Deploy Cloud Foundation Builder VM 

The Cloud Foundation Builder VM is basically a virtual appliance (.ova) that can be downloaded from My VMware.
* The virtual appliance requires 4 vCPU, 4 GB of Memory, and 350 GB of Storage.
* The virtual appliance must be deployed on a suitable platform. This can be on a laptop running VMware Workstation or VMware Fusion or on a dedicated ESXi Host. Its import to note that the virtual appliance must have access to all hosts on the management network.
* The virtual appliance require the following settings during the setup:
	* IP Address
	* Subnet Mask
	* Default Gateway
	* Hostname
	* DNS Servers
	* NTP Servers
	* Password

[![Cloudbuilder](/assets/images/vcf/deploy/cloudbuilder.png "Cloudbuilder"){: .full}](/assets/images/vcf/deploy/cloudbuilder.png)

#### 2. Fill up the configuration excel sheet

Once the Cloud Foundation Builder VM is deployed, you can access the appliance via a web browser with the IP that you have assigned. This should bring you to a welcome page where you can proceed to log in and download the **"Deployment Parameter Sheet"**. 

Do note that the **"Deployment Parameter Sheet"** tend to receive changes every version so its of best interest that you obtain the sheet from the appliance VM itself to ensure compatibility.

<figure class="align-center">
  <a href="/assets/images/vcf/deploy/welcome.jpg"><img src="/assets/images/vcf/deploy/welcome.jpg" alt="Welcome"></a>
  <figcaption style="text-align: center;">Deployment Parameter Sheet Download Page</figcaption>
</figure> 


#### 3. Validate Excel Sheet Configurations

This is a time where it puts all the preparation work that you have done to the test. The validation tool will be able to tell if the configurations that you have inputed into the file tallies to whatever that you have set up beforehand. If there are issues, it will be flagged out with the appropriate error messages and you can then proceed to fix the issue and re-run the validation again. 

<figure class="align-center">
  <a href="/assets/images/vcf/deploy/configuration-valid.jpg"><img src="/assets/images/vcf/deploy/configuration-valid.jpg" alt="Config Validation"></a>
  <figcaption style="text-align: center;">Successful Validated Configurations</figcaption>
</figure> 

#### 4. Sit Back and Relax!

Now that the configurations have been successfully validated, you may now click 'Next' and watch the entire setup take place. Do note there may be errors along the way due to configurations issues and its best to monitor it time to time to ensure the setup takes place. If all goes well, you should see something similar to the screenshot shown below.

<figure class="align-center">
  <a href="/assets/images/vcf/deploy/sddc-success.png"><img src="/assets/images/vcf/deploy/sddc-success.png" alt="SDDC Success"></a>
  <figcaption style="text-align: center;">Successful SDDC Setup</figcaption>
</figure> 

You may now login to the SDDC Manger with the URL that you have configured in the excel sheet. Upon login, it should bring you to the SDDC Manager dashboard page.

<figure class="align-center">
  <a href="/assets/images/vcf/deploy/sddc-manager.png"><img src="/assets/images/vcf/deploy/sddc-manager.png" alt="SDDC Manager"></a>
  <figcaption style="text-align: center;">SDDC Manager Dashboard</figcaption>
</figure> 


Documentation to this section can be found [here](https://docs.vmware.com/en/VMware-Cloud-Foundation/3.0.1/com.vmware.vcf.ovdeploy.doc_301/GUID-F2DCF1B2-4EF6-444E-80BA-8F529A6D0725.html). 

## Key Learnings
Here are some learnings that I took away when deploying with VCF.

### DNS
[![Its Always DNS](/assets/images/vcf/itsalwaysdns.png "Its Always DNS"){: .align-center height="80%" width="80%"}](/assets/images/vcf/itsalwaysdns.png)


**DNS, DNS, DNS!** I cant stress on this enough. Always check if the both forward & reverse DNS to all the components listed in the excel configuration file is working properly.

### NTP
Make sure NTP is properly sync'ed across all hosts and most importantly your Cloud Foundation Builder VM.
- For ESXi host, upon configuring the NTP server on the hosts ensure that the NTP is sync'ed after configuration. If its not, simply restarting the NTP service should suffice.
- For the Cloud Foundation Builder VM, the NTP service should sync with the NTP server that you have configured during the set up. However, I have face issues where the NTP is way off. I manage to fix via restarting the NTP service of the appliance by entering either through the appliance or via ssh and running the command ``systemctl restart ntpd``. Follow this up by restarting the appliance VM and the NTP should sync up perfectly.  

### Password Complexity
Passwords used for all components have to been to be "Strong". It is mention in the documentation that all password used must be a minimum of 8 characters and include at least one uppercase, one lowercase, one digit, and one special character. 

A great example would be the VMware's default password of **"VMware1!"**.

### Network
- Do note that there should be only 2 physical NIC cards presented to the host as VCF 3.0.x currently only supports 2 physical NIC's per ESXi Host and plus one BMC NIC for out-of-band host management. for each host where there must be at least 1 left unassigned to be used for VDS during the setup.
- Ensure that one physical NIC is configured and connected to the vSphere standard switch while leaving the second physical NIC untouched.
- Network should be able to handle jumbo frames connectivity. Use the vmkping command to test. For example. ``vmkping -I vmkX x.x.x.x -d -s 1600``

### vSAN

One major thing about vSAN to take note is to ensure that your caching and capacity disk are eligible for vSAN. Do use the ``vdq -q`` command on the ESXi host to check if the respective disk identifiers displays "Eligible for vSAN". 

If you are keen to know more about this ``vdq`` command you can check out this post [here](https://www.virtuallyghetto.com/2014/02/vdq-useful-little-vsan-utility.html) by William Lam.

During the bring up process, there was an error with regards to setting up vSAN. It complained that it was unable to delete partitions on the host which was very puzzling as the disks within the host was cleared and ``vdq -q`` commands was showing the respective disks meant for vSAN as "Eligible for vSAN".

[![vSAN Error](/assets/images/vcf/vsanerror.jpg "vSAN Error"){: .full}](/assets/images/vcf/vsanerror.png)

We actually took the logs to the engineering team and found out that it was due to a really stupid mistake made in the **"Deployment Parameter Sheet"**. During the naming of the vSAN datastore we named the datastore "vSAN Datastore" which consisted of a space and that spacing was causing issue. Once we remove the space everything ran as expected. So do take note of using spaces for names in general.


## Final Words

There are a couple more stuff that I would like to follow up with on this VCF set up such as:
- Commission/Decommission host
- Expanding the workload domain
- Bringing up the VI workload domain
- Patching with SDDC Manager

In fact, most of it can be explored with VMware's [Hands-On-Lab](https://labs.hol.vmware.com/HOL). So if you are ever interested in the benefits of what VCF can provide from a day 2 perspective do feel free to check out **HOL-1946-01-SLN - VMware Cloud Foundation 3.0 – Getting Started** lab!

Lastly, huge thanks to a senior of mine who got me involved in this VCF setup. 

