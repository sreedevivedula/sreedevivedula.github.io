---
layout:     post
title:      "Getting started with Chef"
subtitle:   "A Basic Tutorial"
date:       2014-10-29
author:     "Sreedevi Vedula"
header-img: ""
---

Chef is a widely used Configuration Automation Framework. In this post, I am documenting the steps that I followed for getting started with Chef. This post covers Chef Component Installation and applying starter cookbook to a Chef node.

<h2 class="section-heading">Chef Workflow</h2>

Chef Workflow is achieved by three major components <b>Chef Server</b>, <b>Chef Nodes</b>, <b>Chef Workstation</b>

(Please note that in the below lines, Machine refers to either a physical machine / a cloud hosted server / virtual image )

<h3>Chef Server</h3>

Machine which stores Chef Cookbooks (Configuration Management Modules), information about the infrastructure like nodes, environments, roles etc. Chef Server is the central hosting server for all the infrastructure configuration.

<h3>Chef Clients / Nodes</h3> 

Machines which form the infrastructure and which are managed by Chef Server. These machines do as minimal work as running a chef client while the entire infrastructure information is hosted on Chef Server.

<h3>Chef Workstation</h3>

Chef Workstation is the machine from which the infrastructure code is developed. This is the development environment for cookbook authors.

<a>
    <img src="{{ site.baseurl }}/img/chef-arch.jpg" alt="Chef Architecture">
</a>

<span class="caption text-muted">Chef Components' Interaction.</span>

Chef comes with a very powerful tool <b>“knife”</b> that orchestrates communication between the Chef Workstation, Chef Server and Chef Clients.

<h3>Choice of Configuration</h3>

To get started with Chef, firstly we need to have two suitable servers / virtual images ready for Chef Server and Chef Node. Your machine can serve as a Chef Workstation. Please see the System requirements in Chef Documentation to learn which configurations can be used for Chef Server, Workstation and Nodes. 

I went for the below configuration.
	
	Chef Server – CentOS 6.4 (64 bit)
	Chef Client / Node – Ubuntu 12.0.4 (64 bit)

We will see how to setup Chef Server, Chef Workstation and Chef Node in that order. Finally we will verify whether the Chef Workflow is setup correctly.

<h3>Setting up Chef Server</h3>
<p>
<ul>
<li>Download Chef-Server from the Chef Server Downloads page. If you are going with a RedHat based VM image , the below is a sample script file you could use to bootstrap Chef Server from a Vagrant VM.
</li>

<li>Place chef-server rpm downloaded in Step 1 in the VM directory. Create a file bootstrap.sh with the below shell script and place it in the Vagrant VM directory. Replace the ip "192.168.33.10" with the VM ip.
	<br>
	<div class="highlight">
    bootstrap.sh

    hostname chefserver.mifosx.com
    echo '192.168.33.10 chefserver chefserver.mifosx.com' >> /etc/hosts
    rpm -ivh /vagrant/chef-server-*.rpm
    chef-server-ctl reconfigure
    chef-server-ctl install opscode-manage
    opscode-manage-ctl reconfigure
    </div>

Note: If there is any issue in executing the commands above, try them one by one in the Vagrant VM and make sure all the commands are successful. Some times there could be a Yum lock held up by other processes. In such cases, kill the processes holding up the yum lock and retry the command.
</li>

3) In the Vagrantfile, add the below line to bootstrap Chef Server on "vagrant up"

    config.vm.provision "shell", path: "bootstrap.sh"

4) Bootstrap Chef Server by running "vagrant up" and make sure Chef Server is up at https://192.168.33.10 from Chef workstation.

We have successfully setup Chef Server. The URL https://192.168.33.10 takes you to the Chef Management Console from which you can create an admin user and an organization.

Configuring Chef Server

After setting up Chef Server, we should create an admin user and create an organization. These steps can be done from the management console following the user-friendly prompts.

Once the organization is setup and a user is created, login to the Chef Management Console and download the Starter Kit in your Chef workstation (which could be your local machine / any other machine set aside to serve as Chef Workstation). This step is very important as this starter kit establishes the connection between your chef workstation and the chef server.