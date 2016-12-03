# CentOS Server Template

Instructions for configuring a CentOS minimal instance to serve as a template for provisioning VMs to host server software (e.g, Nexus, SonarQube, Jenkins, or applications).

This document contains annotated instructions offering reasons for various choices in the configuration. If you don't need the explanation and just want a cheat sheet, read [this document](http://config_steps.md) instead.

## What is this about?

It's becoming the norm to manage server infrastructure dynamically. Resources such as server instances and containers are routinely created and destroyed, with infrastructure tooling in place to ensure user sessions, files/databases, and logs are kept consistent as server instances come and go, and to manage DNS changes dynamically. 

Typically, server characteristics are defined declaratively in configuration files maintained under version control. The configurations are interpreted by tools such as Ansible, Puppet, and Chef to produce deployable VM instances that reliably conform with organizational standards. 

But it isn't practical to build every server instance from source. Instead, people prepare _templates_ of the servers and use those as the basis for provisioning deployable instances. To "bake" a template, you start with a standard iso (or a stripped-down one) and partially provision it. Then it can be used to spin up any number of servers of a given type. 

You might have a standard template for application servers, one for database servers, and one for web servers. Particular instances may differ in some details, like main memory size and number of cores, but are otherwise identical. 

But before you can use a template to spin up VMs automatically, _someone_ has to prepare the template. This is a manual procedure. These instructions are a guide for preparing a generic server template based on CentOS. The procedure for any Fedora-type Linux is the same. 

## Base OS

Use the Minimal ISO as the basis for this template. You can download it from https://www.centos.org/download/.

Use any virtualization tool to instantiate an OS instance from the iso. Accept all defaults. There is no need to tweak any configuration settings, such as main memory size or virtual disk size, as you will do that when you provision server instances based on the template.

### Root password

Define a root password suitable for your needs. It can be "known" within your organization, as the result of this configuration will not be a fully-provisioned server instance, but only a reusable template. Root credentials for deployable instances can be managed in whatever manner is appropriate for your organization.

### Missing packages

Note that the default installation will mention certain packages are not present:

- gcc
- kernel-devel
- patch

Proceed with the installation without these packages. You can install them for particular servers later if necessary. In general, development tools should not be present on server instances, as they provide a potential vector for security exploits.

### Apply system updates

System updates may have been released since the time the iso file was generated. Bring the template instance up to date with this command:

```shell
su -c 'yum update'
```

Depending on the strategy you use to manage VMs in your cloud or cloud-like environment, you may wish to have servers update themselves on a schedule. If so, follow these instructions to configure ```cron``` to run an update script: https://www.centos.org/docs/5/html/yum/sn-updating-your-system.html 

If you are using an _immutable server_ strategy, then there is no need to configure automatic system updates. With that strategy, you will only ever update the templates; deployable instances will all be locked to a selected version and patch level. Updating them means updating the template and saving it as a new version (preserving the old version), and regenerating the deployable instances.

### ssh support

Depending on the strategy you use to manage VMs in your cloud or cloud-like environment, you may or may not want servers to have ```sshd``` support. 

For example, if you use a _push_ strategy to poll and update VMs, some sort of orchestration software will probably connect to each VM using ```ssh```. If you use a _pull_ strategy, then the VMs will reach out across the network on their own for updates and to coordinate with other nodes; no external system needs to know the root password or connect via ```ssh``` with that strategy.

The CentOS Minimal distro has package ```openssh-server``` installed, but not configured to start automatically.

If you _want_ ```sshd``` to start automatically, follow the steps documented here to enable and configure it: https://www.cyberciti.biz/faq/how-to-installing-and-using-ssh-client-server-in-linux/

If you _do not want_ ```sshd``` installed on the template, remove the preinstalled packages:

```shell
yum erase openssh-server
```

If you are not sure whether ```ssh``` packages are installed, find out with these commands:

```shell
yum list installed openssh-server
yum list installed openssh-clients
```

### Software to download components

CentOS Minimal comes with ```curl```, but not with ```wget```. If you need ```wget``` for some of your provisioning steps, install it with:

```shell
su -c 'yum install wget'
```

In most cases, all you will need is a package manager such as ```yum``` and a command-line tool such as ```curl```. If you need anything else to perform scripted installation of components, install those assets on the template so they will be available for all server builds based on that template.

Note that it's a bad idea to install development tools on servers. It's preferable to prepare all necessary assets in advance and install executables. 

## Define any special userids, etc.

If there are special userids or other configuration settings that all your servers must have, then configure them in the template so they will be in place for all deployable VMs based on the template.

## Install/configure additional software

Now tweak the configuration of the template so that it contains all the packages necessary to serve as a base for provisioning deployable servers. The details will depend on your needs, but these instructions provide general guidance.

### End of instructions for the base template

## Provision more-specific templates

Based on the generic server template, you can bake templates for specific categories of servers. Instantiate a VM based on the generic template and install additional components to make it function as a type of server, such as an app server, web server, or database server. Then save the result as another template; one you can use to spin up deployable VMs of each type.

You can take this approach a step further and provision templates for specific applications in your environment. These are useful not only for provisioning production servers, but also for development and testing. Development teams can use them to spin up test environments on demand, with confidence that the test and production configurations are the same.

## What about development and test instances?

You can use this sort of strategy to spin up development VMs, as well. In that case, you will want to install build tools and a version control system (VCS) client, so that developers can check code in and out and run builds. You would probably begin with an iso for a desktop distribution rather than a minimal server distribution. These instructions don't cover development instances, but it is feasible to manage them dynamically just as we do production servers, and feasible to enable development teams to spin up their own development and test environments on a self-service basis as needed. 

## Version the templates

It's customary to maintain multiple versions of templates in case the need arises to support older OS versions or patch levels. Different applications have different dependencies, and not all home-grown or third-party application software is compatible with the latest versions of system components. So, we often need to keep multiple versions of VM templates around.

VCS products are generally geared for storing text files, like program source files and configuration files. They can usually handle small binary files well, such as spreadsheets and diagram files containing system documentation. 

A VCS might not handle large binary files gracefully. In that case, it may not be feasible to store your VM templates in the same VCS as other artifacts. The reason for this is usually that checking out a large binary file takes a long time, and not that the VCS physically can't store the file.

If you can't use a conventional VCS for the VM templates, you will want to establish a naming convention that indicates the version number in the file or directory name. 

The templates may be stored in a package repository or some other shared facility suitable for managing large binary files.

You might have filenames that look something like this:

- rhel-server-base-1.0.5
- rhel-server-base-1.1.0
- windows-server-base-1.0.0
- solaris-server-base-1.4.3
- rhel-appserver-base-1.1.0
- rhel-appserver-base-1.1.1-donotuse
- rhel-appserver-base-1.1.2
- rhel-webserver-base-1.2.6
- rhel-webserver-base-1.2.7
- rhel-webserver-base-1.2.7-170315a
- windows-webserver-base-2.2.1
- windows-webserver-base-2.2.2
- solaris-oracle-base-1.0.4
- solaris-oracle-base-1.1.0-dev
- rhel-billing-app-server-14.9.1
- aix-db2-server-base-3.8.6

and so on. If you needed to spin up a group of Windows Server instances to support an ASP.NET application, you could probably guess which of these templates to use based on the filename.

Note that the template can be used to provision a physical server as well as a VM, so the fact that some of your legacy applications can't (or just don't) live on VMs doesn't preclude the use of template instances or dynamic infrastructure practices in general.

The last two filenames in the list suggest special one-off configurations to support unique cases. 

The name ```rhel-billing-app-server-14.9.1``` suggests the template pertains to a particular application. Perhaps this application has some outdated dependencies and won't run on a standard configuration. You can still manage it dynamically. (In many shops people assume they must manage these one-off cases manually.)

The name ```aix-db2-server-base-3.8.6``` suggests the template is used to provision AIX instances to suport DB2. Many organizations use DB2, but most of them don't run AIX as VMs under IBM PowerVM. Their VM infrastructure supports Intel and compatible processors only. So, the AIX instance may be built on a physical box. You can still manage the configuration dynamically. (In many shops people assume dynamic infrastructure management techniques apply strictly to VMs.)



