architecture-setup
==================

architecture-setup is used by [pancancer\_launcher](https://github.com/ICGC-TCGA-PanCancer/pancancer_launcher/) to create a docker image that can be used to create and manage a fleet of VMs that will run SeqWare workflows.

##Submodules of architecture-setup:

* [monitoring-bag](https://github.com/ICGC-TCGA-PanCancer/monitoring-bag.git) - This is used to set up monitoring components on the main controller of the fleet, and also on the worker nodes.
* [central-decider-client](https://github.com/ICGC-TCGA-PanCancer/central-decider-client) - This is used on the main controller node to get INI files for the workers.
* [container-host-bag](https://github.com/ICGC-TCGA-PanCancer/container-host-bag) - This is used to set up the worker nodes.

This project contains ansible roles that are used to create the pancancer\_launcher container.

 - bindle-profiles - This is a basic bootstrap role that ensures that the submodules are checked out correctly, and then places some of the pancancer\_launcher scripts on their correct paths.
 - arch3_master - This sets up Architecture3 components inside the pancancer\_launcher container
 - java - This is a role that is used to install and set up Java inside the pancancer\_launcher.

The scripts and files that you will most likely be using or modifying are:

 - **start\_launcher\_container.sh** - This is used to start an instance of pancancer\_launcher. The script is called like this:

        bash start_launcher_container.sh -p ~/.ssh/my_key.pem -i 3.1.6 -e AWS -t true -f testFleet

The arguments are:

\-p, --pem_key - The path to the pem key file you want to use to start up new workers.

\-i, --image_version - The version of pancancer_launcher you want to run.

\-e, --host_env - The host environment you are running in (Either "AWS" or "OPENSTACK"). If you do not specify a value, "AWS" will be defaulted.

\-t, --test_mode - Run in test mode (lauches workers immediately when container starts). Defaults to "false"

\-f, --fleet_name - The name of the fleet of workers that will be managed by this launcher. If you do not specify one, a random name will be generated.

--target_env - Only used when running in test mode.

\-h, --help - Prints a help message.

  It is also possible to call this in a simpler way by omitting the last three arguments. They will take on the assumed values "false" and "AWS", and the fleet name will be a randomly generated string:

      bash start_launcher_container.sh -p ~/.ssh/my_key.pem -i 3.1.6

 - **roles/bindle-profiles/files/bashrc** - This is a bashrc file that is set up for the ubuntu use inside the pancancer\_launcher container. It is based on the Ubuntu 14.04 default bashrc file, but has a custom prompt containing the pancancer\_launcher version number.
 - **roles/bindle-profiles/files/launch\_workers.sh** - This is a file that can be used to launcher worker nodes automatically from the pancancer\_launcher container. It is primarily intended to be used from Jenkins or other build/test tools so that a complete end-to-end test can be executed without manual intervention. It is activated by passing the value "true" (to indicate that you want to run in test mode) as the last argument to `start_launcher_container.sh`.
 - **start\_services\_in\_container.sh** - This script runs when the container starts up. It is responsible for starting up the services inside the container: rabbitMQ, redis, postgres, sensu, and uchiwa. It also sets up the rabbitMQ users and vhosts (if they are not set up already), and sets the `PUBLIC_IP_ADDRESS` and `SENSU_SERVER_IP_ADDRESS` variables that are needed inside the container.
