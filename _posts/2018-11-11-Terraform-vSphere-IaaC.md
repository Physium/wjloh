---
title: "Terraform with vSphere"
categories: 
  - IaaC
tags:
  - Terraform
  - vSphere
toc: true
---

Over the weekend instead of accomplishing my entire TO-DO list, I kinda procastinated and went totally of the track instead. I've have always wanted to check out what this whole "Infrastructure as a Code" thing is all about and I have heard quite abit about Terraform being one of the solution that can help achieve this. 

So with Terraform, you can make calls to vSphere to provision VMs (Virtual Machines) without the need of accessing the vSphere/vCenter Client interface. All this is done via writing your own configuration file with Terraform's own configuration language, HCL.

## Getting Started

Considering that I'm on macOS, getting started is as easy as:
```sh
brew install Terraform
```

Homebrew is basically a package manager for macOS which is similar to the linux variations of apt-get, yum and what not. If you are not using homebrew for Mac, I highly suggest you do consider getting it as it greatly simplifies your developer environment setup.

If you are on Windows, I believe Windows now has a package manager called **[Chocolatey](https://chocolatey.org/)** where you can simply run:

``` posh
choco install Terraform
```

## The Basics

So there are 3 components to 

## Setting up Terraform for vSphere

I decided to start of with one of the tempate that they have provided **[here](https://www.terraform.io/docs/providers/vsphere/index.html#argument-reference)]** but only to find out that they dont exactly work.

