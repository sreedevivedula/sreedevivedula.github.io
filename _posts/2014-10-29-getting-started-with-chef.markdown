---
layout:     post
title:      "Getting started with Chef"
subtitle:   "A Basic Tutorial"
date:       2014-10-29
author:     "Sreedevi Vedula"
header-img: ""
tags: DevOps, Chef
---

Chef is a widely used Configuration Automation Framework. In this post, I am documenting the steps that I followed for getting started with Chef. This post covers Chef Component Installation and applying starter cookbook to a Chef node.

<h2 class="section-heading">Chef Workflow</h2>

Chef Workflow is achieved by three major components <b>Chef Server</b>, <b>Chef Nodes</b>, <b>Chef Workstation</b>

(Please note that in the below lines, Machine refers to either a physical machine or a cloud hosted server or virtual image )

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

<h2 class="section-heading">Setting up Chef Server</h2>

Download Chef-Server from the Chef Server Downloads page and follow the instructions in the documentation.
After successfully setting up Chef Server, create an admin user and an organization in the Chef Management Console.

<h3>Configuring Chef Server</h3>

Once the organization is setup and a user is created, login to the Chef Management Console and download the Starter Kit in your Chef workstation (which could be your local machine / any other machine set aside to serve as Chef Workstation). This step is very important as this starter kit establishes the connection between your chef workstation and the chef server.

<h2 class="section-heading">Setting up Chef Workstation</h2>

Let us now move on to setting up Chef Workstation. I have Mac OS X (10.9.5) and I used the same as my workstation. However, the installation should not be different for any Unix / Linux based systems.

There are two approaches for installing Chef Workstation

1) Using omnibus installer
2) Using Chef Developer Kit

I have faced issues while using omnibus installer during installation of some dependencies, so I have switched to using Chef Developer Kit.

After installing Chef Developer Kit, verify the installation using chef-verify and set system ruby as mentioned in the instructions. At the end of this, if you run "which ruby", it should point to /opt/chefdk/embedded/bin/ruby. It is important to make sure that we are using embedded ruby while working with chef as it contains the right version and has some pre-required gems installed.

<h3>Setting up Chef Repo in the Chef Workstation</h3>

The starter kit you downloaded earlier explodes into a chef-repo which is the central repo for all your infrastructure code.

<h3>Chef-Repo</h3>

Chef-Repo is the directory on your workstation which is the repository for your Chef work.

It contains the powerful knife tool and a sample cookbook. Also contains .chef directory which holds configuration information. The Chef Server configuration is stored in .chef/knife.rb

This repo will be the place for your infrastructure code, so please set up a git repository for this repo.

<h3>knife</h3>

Knife tool handles Chef Workflow Management. It uploads cookbooks from workstation to Chef Server, manages nodes, stores run lists for nodes etc. We will see some of its most used operations in an example in later posts. Run "knife help" to get an overview of it's capabilities.

<h2 class="section-heading">Setting up a Chef Node</h2>
Let us now setup a Chef node and bootstrap it using knife. Create a Vagrant VM from any image of your choice and make sure that the VM is able access Chef Server. The below is a sample shell script that could be used while starting Vagrant VM.Please note that "192.168.33.10" is the IP of Chef Server and "192.168.33.11" is the IP of the Chef node.

{% highlight html %}
hostname node1.chef-demo.com
echo '192.168.33.10 chefserver chefserver.chef-demo.com' >> /etc/hosts
echo '192.168.33.11 node1 node1.chef-demo.com' >> /etc/hosts
{% endhighlight %}

Once the Vagrant VM is brought up, we can bootstrap the node with the help of knife.

<h3>Bootstrapping Chef Node with knife</h3>

Bootstrapping of a node by knife installs Chef Client software on the node, generates client key and saves it to the node.

In the below command "node1" is the name of the Chef node which will be used for referring to this machine in Chef Workflow and -x option should be given the root user name and -P option should be given the root password. The below command should be run in chef-repo.

{% highlight html %}
knife bootstrap 192.168.33.11 -x root -P vagrant -N node1
{% endhighlight %}

<h3>Points to Note</h3>

The above bootstrap command may throw an error if the node's fingerprint is already stored in the Chef Workstation. Open ~/.ssh/known_hosts file and remove the entry for the node's IP and retry.

Make sure that the workstation and node are able to access chef server with the hostname. If not, edit /etc/hosts and set the host name.

We are setup with all the components in the Chef Workflow and are ready to verify the whole setup.

<h3>Verifying Chef Setup</h3>

Let us do a quick check of whether the Chef workflow is setup correctly.

Go to chef-repo directory and run the below command to look up the cookbooks available in the project.

{% highlight html %}
knife cookbook list
{% endhighlight %}

If the above command throws an error that you cannot contact Chef Server, please edit /etc/hosts on your Chef workstation and add an entry for chef server as below

{% highlight html %}
192.168.33.10 chefserver
{% endhighlight %}

Now, re-run the "knife cookbook list" command and verify that no cookbooks are yet uploaded to the Server

Now, upload the starter cookbook using knife
{% highlight html %}
knife cookbook upload starter
{% endhighlight %}

Verify that the cookbook is uploaded to Chef server by running "knife cookbook list"
{% highlight html %}
knife cookbook list
{% endhighlight %}

Now, we need to add the starter cookbook to our Chef node's run_list. A run_list is a series of recipes that define the configuration policy for a Chef node. Please note that "node1" is the name of Chef node that we used to register the Chef node machine.

{% highlight html %}
knife node run_list add node1 'recipe[starter::default]
{% endhighlight %}

SSH into chef node and run sudo chef-client to download the cookbook and apply it to the node.

{% highlight html %}
sudo chef-client
{% endhighlight %}

Verify that Chef log output is printed.

Hurray! We are all setup with Chef. Happy Learning!