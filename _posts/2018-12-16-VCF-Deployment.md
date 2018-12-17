---
title: "Up & Running with VMware Cloud Foundation(VCF)"
categories: 
  - SDDC
tags:
  - VMware Cloud Foundation
  - VCF
toc: false
---
After much procrastination, I decided to set sometime aside during this short December break of mine to write about this topic. Recently, I had the chance to get my hands on some enterprise grade hardware to give VCF (VMware Cloud Foundation) a shot. There has been much talks about VCF being the foundation of SDDC and how it to work across the different cloud offerings creating a seamless hybrid approach. This sits in well with the recent collaboration announcements of VMware and AWS where VCF is the foundation for public cloud offerings in particular VMware on AWS and AWS outpost.

## VCF Benefits

## Deploying VCF

The entire bring up process with VCF is pretty straightforward. What's most important is getting the necessary preparation work done in order for the Cloud Foundation Builder VM to work its magic. I wont go into the details of that as there's an entire documentation on the planning and preparation required for the setup to take place. I would suggest giving that a read.

1. Deploy Cloud Foundation Builder VM
2. Fill up excel sheet
3. Validate Excel Sheet Configs
4. Sit Back and Grab a Coffee

## Learnings
Here are some learnings that I took away when deploying with VCF
- DNS,DNS, DNS! I cant stress this enough always check if the DNS to all the components listed in the excel configs is working.
- Passwords used for all components have to been to be "Strong". It is mention in the documentation that all password used must be a minimum of 8 characters and include at least one uppercase, one lowercase, one digit, and one special character. A great example would be the VMware's default password of "VMware1!".
- There should be a minimum of 2 NIC cards for each host where there must be at least 1 left unassigned to be used for VDS during the setup.
- Network should be able to handle jumbo frames. Use the vmkping command to test. ``vmkping -I vmkX x.x.x.x -d -s 1600``
- Ensure that when naming your vSAN datastore in the configs, do not include any form of spaces as this will prevent the set up of 