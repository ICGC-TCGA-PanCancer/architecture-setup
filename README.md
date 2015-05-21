architecture-setup
================

This tool is used to install Bindle and all of its dependencies pertaining to the pancancer project.
The result is a host that can be used to create new SeqWare images with pancancer workflows pre-installed.

###Setting up the architecture-setup server
For AWS, use an m3.large, with Ubuntu 14.04. You will need an 8 GB root partition.

###Sets up the following on the desired node(s)

* [Bindle](https://github.com/CloudBindle/Bindle)
* [Seqware-bag](https://github.com/SeqWare/seqware-bag.git)
* [Pancancer-bag](https://github.com/ICGC-TCGA-PanCancer/pancancer-bag.git)
* [Monitoring-bag](https://github.com/ICGC-TCGA-PanCancer/monitoring-bag.git)
* [workflow-decider] (https://github.com/ICGC-TCGA-PanCancer/workflow-decider.git)
* [youxia] (https://github.com/CloudBindle/youxia)

## Setup

**!!! IMPORTANT !!!**
You will need to get a confidential pem key for GNOS upload/download from your GNOS admin or a fellow cloud shepard. Please copy it to /home/ubuntu/.ssh/gnostest.pem before running the next step, or otherwise the Ansible playbook will fail to run completely leaving the system in a half-broken state.

If you do *not* have a valid GNOS pem key, the setup script will create an empty file for you, but you will need to replace it with a valid key file before setting up any workflows that require it.
<!--
This playbook relies upon bindle's install playbook. You will need a fairly current version of git (at least version 1.9.1)

    sudo add-apt-repository ppa:git-core/ppa
    sudo apt-get install git
    git clone https://github.com/ICGC-TCGA-PanCancer/architecture-setup.git
    cd architecture-setup 
    bash setup.sh
-->
!!! IMPORTANT !!!    
You will also need to get a confidential pem key for GNOS upload/download from your GNOS admin or a fellow cloud shepard. Please copy it to /home/ubuntu/.ssh/gnostest.pem before running the next step, or otherwise the Ansible playbook will fail to run completely leaving the system in a half-broken state.

The simplest way to set up the machine you are *currently* logged into as a launcher is to run the `setup.sh` script. You can do this in a single step, like this:

    cd ~
    curl -L https://github.com/ICGC-TCGA-PanCancer/architecture-setup/releases/download/2.0.0/setup.sh | bash

The above command will download and execute the setup script for architecture-setup version 2.0.0. If you want to install architecture-setup based on the current develop branch, use this command instead:

    curl -L https://raw.githubusercontent.com/ICGC-TCGA-PanCancer/architecture-setup/develop/setup.sh | bash

Sometimes, parts of the main playbook may fail. If this happens, you may want to try making any changes (if necessary - if it's simply a download that failed, as often happens with the vagrant plugins, you probably don't need to change anything), and re-running the playbook:

    cd architecture-setup
    ansible-playbook -i inventory site.yml

If for some reason you need to perform a more manual setup, or you wish to apply the script to an existing launcher, clone this repo and then move the setup script to the *parent* directory before executing it.

<!-- ## Setting up a different host as launcher 

If you wish to setup some other host as the launcher, you will need to edit your inventory. Replace the pem file and the ip address of the launcher host that you wish to create with your desired launcher host. Ansible will obviously require SSH, therefore make sure that port 22 is open to your desired launcher host.
        
    ansible-playbook -i inventory site.yml
-->

Now follow the rest of the pancancer-info instructions on how to setup and use bindle

***

## Upgrading to a new version

***FOR PRE-1.0.7 LAUNCHERS:***
New scripts were added to architecture-setup 1.0.7 to make upgrading to newer version easier. If your launcher has a version that is older than 1.0.7, you should follow these instructions to catch it up:

1. Download this script: https://github.com/ICGC-TCGA-PanCancer/architecture-setup/releases/download/1.0.7/update_archsetup_to_1.0.7.sh to your launcher.<br/>
It will make a backup copy of your old `~/architecture-setup` directory, and then create a new one and check out architecture-setup 1.0.7. <br/>
Execute the script as `bash update_archsetup_to_1.0.7.sh`.
2. `cd ~/architecture-setup`
3. `bash setup.sh`
4. `ansible-playbook -i inventory site.yml`
5. `bash upgrade_launcher_and_workers.sh 1.0.7`

Steps 1 through 4 will bring your architecture-setup up to date. Step 5 will upgrade your worker nodes.

***FOR LAUNCHERS WITH VERSION 1.0.7 AND GREATER:***

If you have a launcher that has an older version of architecture-setup and you wish to upgrade, you can upgrade your launcher using the scripts available:

 - `upgrade_architecture_setup.sh` This script will upgrade your architecture-setup to a specific release version. It will also re-run the architecture-setup ansible playbook.
 - `get_nodes_for_update.sh` This script will get a list of nodes whose last Sanger workflow run completed successfully. An inventory file will be built, and this file can be used to upgrade the worker nodes.
 - `upgrade_worker_nodes.sh` This script will download a specific version of the Sanger workflow and use the inventory file generated by `get_nodes_for_update.sh` to upgrade the worker nodes.
 - `upgrade_launcher_and_workers.sh` This script will execute all three scripts above. This script is intended to be used for a complete end-to-end launcher upgrade. If you only need to run certain parts of the process, you can call the other scripts explicitly.

**NOTE:** These scripts assume that your directory structure looks like this:
<pre>
  ~/.bindle
  ~/architecture-setup
    /Bindle
    /monitoring-bag
    /pancancer-bag
    /seqware-bag
    /workflow-decider
    /youxia
</pre>

###upgrade_launcher_and_workers.sh

This script will uprade a launcher and all of its workers by running the other three scripts. It takes one argument: a version of architecture-setup to upgrade the launcher to.

It is used like this:

    bash upgrade_launcher_and_workers.sh 1.0.5

You can also choose to run any of the child scripts independently.

####upgrade_architecture_setup.sh

This script will checkout a specific version of architecture-setup. It will update all bindle config files in ~/.bindle/*.cfg with the workflow version associated with the specified architecture-setup version. It will then run the main ansible playbook for architecture-setup, using the updated `roles/bindle-profiles/vars/main.yml`.

Example:

    upgrade_architecture_setup.sh -v 1.0.5
    
The file `roles/bindle-profiles/vars/main.yml` that is a part of architecture-setup 1.0.5 will be downloaded to the current local repository. Ansible will then be run (`sudo ansible-playbook -i inventory site.yml`).

**NOTE:** This script will update *local* repositories and bindle configuration files. It may warn you if you have changes to files tracked in github, but you may want to make backup copies of any bindle config files (`~/.bindle/*.cfg`), and any important changes you need in `~/architecture-setup/Bindle`, `~/architecture-setup/panancer-bag`, `~/architecture-setup/monitoring-bag`, `~/architecture-setup/youxiua`, `~/architecture-setup/seqware-bag`, or `~/architecture-setup/workflow-decider`.

**NOTE:** This script will exit without error if it detects that the local architecture-setup repository is the same version as the one you requested. This script will exit *with* error (code=1) if it detects that there are changes to any of the loca repositories that it normally would try to check out. It will also exit with error if an error occurs while checking out architecture-setup from github.

The output from this script will be in a file named upgrade_architecture_setup.sh.log.

####get_nodes_for_update.sh

This script will query all nodes accessible vagrant (using `vagrant global-status`). It will generate a new inventory file in `~/architecture-setup/pancancer-bag/workflow-update/` based on which nodes report that they are "completed" (meaning that their last Sanger workflow run completed without error and they are not currently running another workflow).

**NOTE:** This script will exit with error (code=1) if there are no vagrant cannot find any worker nodes running. You can verify this manually by executing `vagrant global-status`.

The output from this script will be in a file named get_nodes_for_update.sh.log.

####upgrade_worker_nodes.sh

This script will use the inventory file generated by `get_nodes_for_update.sh` to upgrade worker nodes to a specific workflow version. It will also download the workflow bundle, if it is not already available on the launcher. You can run this script like this:

    bash upgrade_worker_nodes.sh 1.0.5

This will tell the script to download `Workflow_Bundle_SangerPancancerCgpCnIndelSnvStr_1.0.5_SeqWare_1.1.0-alpha.5.zip` if it is not available. The script will then run the workflow-update playbook, in `~/architecture-setup/pancancer-bag/workflow-update`.

The output from this script will be in a file named upgrade_worker_nodes.sh.

***

## Creating a new release

It is possible to create a new release of architecture-setup from here using the script `create_release.sh`. To do this, you will need:

 - bash 4.* (If you are running on OS X, this can be installed via homebrew)
 - jq 1.3 or better
 - A github authentication token file

You will need to install the additional package [jq](http://stedolan.github.io/jq/), at least version 1.3:

    sudo apt-get install jq

If your ubuntu system is unable to get jq using apt-get, the compiled binary can also be downloaded here: http://stedolan.github.io/jq/download/
    
You wil need a github authentication token so that you can make changes to repositories. Detailed instructions can be found [here](https://help.github.com/articles/creating-an-access-token-for-command-line-use/). The token should be saved in a file named **`github.token`** in the `architecture-setup` directory. You will need to ensure that your github account has permission to create releases and check in files on the relevant projects or this script may fail.

Creating a new release involves examining projects that architecture-setup depends on and if there have been any commits since the last release of architecture-setup, the dependencies will be updated with a new release and the `vars/main.yml` file will be updated to refer to these new releases and checked in. A new release of architecture-setup will also be created.

All new releases created by this process are created as *draft* releases. This gives you the chance to accurately update the release notes for each repository's release (the script will only populate the release notes with a simple comment: "generated release"). The releases must be manually published after they have been created.

Executing the script is done like this:

    bash create_release.sh [-t][-h]
    
`-t` is used to execute the script in test mode. When executed in test mode, no releases are created and no files are committed.

`-h` will print help text.

**NOTE:** It is *strongly* recommended that you run this at least once with test mode enabled (set the `-t` flag) to ensure that the results are what you would expect.

The output of this script will be in a file named create_release.sh.log.

##Updating references to the workflow version

To update references to the workflow version in various files, you can use the script `update_workflow_refs.sh`. This script will update files that may contain references to specific workflow versions, but are generic files that should be updated when a new workflow is released. To use this script, you will need a github token file (see the section on "Creating a new release" for more information). Running the script looks like this:

    update_workflow_refs.sh -w 1.0.6

Currently, the files updated by this script are:
 - pancancer-bag/workflow-update/roles/update_workflow/vars/main.yml
 - monitoring-bag/roles/client/vars/main.yml
 - architecture-setup/roles/bindle-profiles/main.yml
 - workflow-decider/conf/sites/decider.oicr.ini
 - workflow-decider/conf/decider.ini

Users should be aware that if thay have hard-coded references to a specific workflow version in other files that are specific to their launcher, they may have to update those references on their own. Please get in touch if you think that there are additional files that should be updated by this process, but are not mentioned here.

**NOTE:** It is *strongly* recommended that you run this at least once with test mode enabled (set the `-t` flag) to ensure that the results are what you would expect.

The output of this script will be saved in a file named update_workflow_refs.sh.log
