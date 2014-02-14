#Newmedia Packer

[Packer](http://www.packer.io/intro) is used to build the virtual machines used at highwire. You can interact with this repository when you need to modify the underlying stack (eg: httpd, php, mysql) for a base box.

Packer is controlled by the ```{ "provisioner": {} }``` section of [centos-5.10-x86_64.json](centos-5.10-x86_64.json). The scripts provisioner executes shell scripts stored in [scripts](scripts). The chef-solo provisioner executes a run_list of cookbooks that are managed by the [Berksfile](Berksfile).

##Initial Setup
1. Download and install the latest [VirtualBox](https://www.virtualbox.org/wiki/Downloads).

1. Download and install [Vagrant](http://www.vagrantup.com/downloads.html).

1. Install required ruby gems [Berkshelf](http://berkshelf.com/) and [Thor](https://github.com/erikhuda/thor/wiki).

    ```bash
    $ gem install berkshelf thor
    ```

1. Download and install [Packer](http://www.packer.io/intro/getting-started/setup.html)

1. Clone the hw-packer repository.

   ```bash
   $ git clone git@github.com:highwire/hw-packer.git ~/hw-packer
   ```

##Creating Boxes
1. [Thor](https://github.com/erikhuda/thor/wiki) provides the command line interface. It has a help system that can provide assistance on the Highwire Press (hwp) commands:

   ```bash
   $ cd ~/hw-packer
   $ thor help hwp
   Commands:
     thor hwp:build           # Build a base vagrant box from chef cookbooks.
     thor hwp:clean [WHAT]    # iso|box|all - downloaded iso files, built virtual boxes, everything
     thor hwp:help [COMMAND]  # Describe available commands or one specific command
     thor hwp:validate        # Validate all the packer templates eg: centos-5.10-x86_64.json
   ```

1. In order to build the base box:

    ```bash
    $ cd ~/hw-packer
    $ thor hwp:build
    ```

1. Now, it is time to wait. The first time that this command is run, it will download the .iso file for the machine and then run through the scripts to set up the machine automatically. During this time, we recommend, relaxation, coffee, or other tasks be compeleted. This can take additional time depending on the internet connection.

1. When the build is complete you will have a box that can be distributed for import into Vagrant. The path to the box is visible in the output of ```thor hwp:build```
    ```bash
    Build 'virtualbox-iso' finished.

    ==> Builds finished. The artifacts of successful builds are:
    --> virtualbox-iso: 'virtualbox' provider box: builds/virtualbox/opscode_centos-5.10_chef-latest.box
    ```

##Using Boxes
In a typical setup, the base box is distributed from a url. In this example, we will target the box generated in the previous step.

1. You will need to Add a built box to vagrant. The box location can be local or remote.
    ```bash
    $ cd ~/hw-packer
    $ vagrant box add centos510 builds/virtualbox/opscode_centos-5.10_chef-latest.box
    ```

1. Perform a one time installation of plugins for vagrant.

   ```bash
   $ vagrant plugin install vagrant-berkshelf
   $ vagrant plugin install vagrant-omnibus
   $ vagrant plugin install vagrant-vbguest
   ```

1. Use a Vagrantfile that references the box you added. An example one [exists in the highwire chef repository](https://github.com/highwire/chef/blob/master/Vagrantfile).

   Bring up a vagrant instance built from our base box.
   ```bash
   $ git clone git@github.com:highwire/chef.git ~/hw-chef
   $ cd ~/hw-chef
   $ vagrant up
   ```
   You should be able to ssh into the new instance after it comes up.
   ```bash
   $ cd ~/hw-chef
   $ vagrant ssh
   Last login: Mon Jan 01 00:00:00 2014 from 10.0.2.2
   [vagrant@localhost ~]$
   ```
   You can provision the sites [defined for highwire chef](https://github.com/highwire/chef/blob/master/infrastructure/jcore.json).
   ```bash
   $ cd ~/hw-chef
   $ vagrant provision
   ```