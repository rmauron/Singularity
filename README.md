# Recipe for Singularity


Date: March 2023

Note from: Raphaël Mauron (<raphael.mauron@gmail.com>)
<br>



## Goals

-   Install Singularity

-   Build a Singularity container

-   Run a Singularity container

-   Run from online repository

Being able to run and share scripts with all their dependencies on your machine or on HPCs.
<br>


## Resources from UPPMAX IT


All you need to do is described on this recipe but if more resources are needed, here you have interesting links.

Guidelines for installing Singularity on your computer: <https://pmitev.github.io/UPPMAX-Singularity-workshop/installation/>

If have accounts and active project on Alvis or Rackham, able to build on the clusters directly: <https://pmitev.github.io/UPPMAX-Singularity-workshop/fakeroot/>

Complete UPPMAX workshop material: <https://pmitev.github.io/UPPMAX-Singularity-workshop/>
<br>


## Why is all this needed?


Virtualisation is the technology which allows to create independent environments from your machine. It has many functionalities, like running various systems or applications on a single machine, sharing resources (e.g. processors, memory, etc.)

Virtualisation allows the creation of virtual machines (VM) which are isolated and autonomous informatic environments with their own operating systems. Without going into the details, this virtualisation is important in the scope of this tutorial because Singularity runs only on Linux systems. We will install and use a VM with Ubuntu in order to use Singularity.

In this recipe, Vagrant is the tool used to create and manage those environments. It allows to create a VirtualBox VM in very simple steps.
<br>


## Singularity


Singularity is a container virtualization tool made for scientific applications and high performance computing. It is compatible with Docker Images but does not allow for administrative privileges which facilitates the management of HPC clusters. Essentially, the waiting queue system is respected with Singularity while it is not with Docker.

A Singularity container is like every other container, hence an isolated operative environment with all the required resources included. Containers are built from images. Images contain application code, libraries, dependencies needed to execute the application.

Singularity containers, once built, can be saved for further run or can be shared. Anyone with access to the container can use it independently on its own system to run the application with the exact same versions of each dependency.
<br>



## Install Singularity on your machine:
    

(for Mac with Intel-64)

  Install homebrew (if not already installed):

<https://brew.sh/>

Or simply:



    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

 

  Install virtualbox, vagrant and vagrant-manager:



    brew  install --cask virtualbox vagrant vagrant-manager

 

(note that for M1 or M2 chips, the process can be different so it is not covered here)

What I know is that VirtualBox can be installed manually (Developer preview for macOS/ Arm64 (M1/M2) that you can find here: <https://www.virtualbox.org/wiki/Downloads>). 

Once VirtualBox installed you can run the following command:



    brew install --cask vagrant vagrant-manager

 



## Create a temporary VM with vagrant for building Singularity containers
    

  Create and enter a directory to be used with your Vagrant VM:



    mkdir  vm-singularity-ce && \\
     cd  vm-singularity-ce

 

(if it is the first time, skip point 1.4)

  Destroy and remove vagrant vagrant file:

If you have already created and used this folder for another VM, you will need to destroy the VM and delete the Vagrantfile:



    vagrant destroy && \\
        rm Vagrantfile

 

  Bring up the virtual machine:

Inside the created new repository, create the Vagrantfile with these steps:



    vim Vagrantfile

 

The following code is a base for the Vagrantfile. Specify the path/to/destination on the hose and the path/from/vm in the vim. 

When it is done, type ":wq" which stands for "write quit".


```
Vagrant.configure("2") do |config|\
  config.vm.box = "sylabs/singularity-ce-3.8-ubuntu-bionic64"

  #share files "path/to/destination", "path/from/vm"

  config.vm.synced_folder "./", "/vagrant"\
end
```
 

Once you are back on the terminal, run to force vagrant to recreate the vm:



    vagrant up --provision

 

If everything ran smoothly, it means you created the VM.

Access the VM with:



    vagrant ssh

 

  Check version installed:


```
vagrant@vagrant:~$  singularity version\
3.8.0
```


 

! Version more recent than 3.7 is required for UPPMAX.

If you reach this point, you are all set up to build your Singularity container.



## Build Singularity container
    

Once in the Singularity VM, you can build a Singularity container.

First change directory:



    cd /vagrant

 

With the example from UPPMAX workshop:

Create the definition file. It is where you specify which tool to use, the version of them, all the dependencies, and so on. Usually, the user do not have all the permissions to create files, hence we use "vi" instead of "vim" in this case. Note that here, you can name your definition file as you wish.



    vi lolcow.def

 

Paste and type ":wq"


```
BootStrap: docker\
From: ubuntu:16.04

%post\
  apt-get -y update\
  apt-get -y install fortune cowsay lolcat

%environment\
 export  LC_ALL=C\
 export  PATH=/usr/games:$PATH

%runscript\
  fortune | cowsay | lolcat
```
 

The definition file used for the tool ArchR software is visible here: <https://github.com/rmauron/Singularity/blob/cee4b188bbec5b85470674b874ad2f534ccc2dce/ArchR/ArchR.def>

Once back on the vagrant terminal, build the container:



    sudo  singularity  build lolcow.sif lolcow.def

 

If everything goes smoothly, you should now have the .def and .sif files. The .def is the definition file where we specified all the dependencies. The .sif is the actual container that we want to run.



##Run Singularity container
    

With the Singularity container built, you can run it. Try the lolcow tutorial.



    ./lolcow.sif

 

It should make you laugh.

But here is the idea behind running a Singularity container.



## Stop and destroy the VM
    

You can stop the VM with:



    vagrant halt

 

You might want to destroy the VM. You can do it with (after having stopped it beforehand):



    vagrant destroy -f

 



## Prerequisites for running graphical programs remotely with Xquartz
    

In case of R applications, graphical desktops are more handy. For it, follow the instructions below. This is required to run the ArchR container. <https://www.cyberciti.biz/faq/apple-osx-mountain-lion-mavericks-install-xquartz-server/>

Or follow those instructions:

Install Xquartz to allow X11 forwarding:



    brew install --cask xquartz

 

Reboot mac:



    sudo reboot

 


## Run Singularity container on HPC
    

You have to enable the X11 forwarding so we use the "-Y" command when connecting to the HPC:



    ssh -Y user@server-ip

 

Or more practically when connecting to UPPMAX:



    ssh -Y username@rackham.uppmax.uu.se

 



## Run the ArchR Singularity container on UPPMAX
    

From the entry page a.k.a your user directory, run:



    singularity run /tmp/ArchR-20.04.sif

 

The XQuartz window should open the RStudio desktop where you can run any script but with the packages (version dependent) needed for this specific application.


