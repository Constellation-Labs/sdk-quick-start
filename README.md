## Welcome to Euclid Development Environment
# Dependencies
## Docker
* You should have Docker installed
* Check the [installation guide](https://docs.docker.com/engine/install/)
* You need to have **at least 8GB of RAM allocated to Docker**

## Cargo
* Cargo is the Rust package manager
* [Here](https://doc.rust-lang.org/cargo/getting-started/installation.html) you can check how to install Rust and Cargo

## Ansible
* Ansible is a configuration tool for configuring and deploying to remote hosts
* [Here](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) you can check how to install Ansible

# First Steps

## Understanding the folder structure

With the euclid-development-environment cloned, you'll see the following structure
```
- infra
  - ansible
  - docker
- scripts
  - custom-template.sh
  - docker.sh
  - hydra
  - hydra-update
  - join-cluster.sh
  - utils.sh
- source
  - metagraph-l0
    - genesis
  - global-l0
    - genesis
  - p12-files
  - project
- euclid.json
```
let's see what each of these directories represents:

### Infra
This directory contains infrastructure related to the running of Euclid. 
- Docker: This directory contains docker configuration, including Dockerfiles, and port, IP, and name configurations for running Euclid locally. 
- Ansible: This directory contains Ansible configurations used for configuring and deploying to remote hosts. 

### Scripts
Thats the "home" of hydra script, here you'll find the `hydra` and `hydra-update` scripts

### Source
Here is the home of the local codebase, this directories will be filled in ways you build the containers.

Example: let's say that you'll build the container `metagraph-l0` (it will be explained better below, don't worry), on the directory `source/metagraph-l0` will be created one folder with the local codebase of the `metagraph-l0` node

The example above applies to the other containers: `metagraph-l1-currency`, `dag-l1`, `global-l0`

Inside the source folder, we also have the sub-directory `p12-files`. In this directory, you can provide the custom `.p12` files. This directory already comes with some examples, but they can be overwritten/removes to your own files.

### hydra.cfg
Here is the hydra configuration file, there you can set the `p12` file names and your GITHUB_TOKEN. It's required to fill the GitHub token here to run the `hydra` script

## Hydra scripts options
Run the following command to list all the possibilities of the `hydra` script

```
./hydra -h
```

you should see something like this:

```
USAGE: hydra <COMMAND>

COMMANDS:
  install           Removes the remote git
  build             Build all the containers
  start_genesis     Start containers from the genesis snapshot (erasing history)
  start_rollback    Start containers from the last snapshot (maintaining history)
  stop              Stop all the containers
  destroy           Destroy all the containers
  status            Check the status of the containers
  remote_deploy     Remotely deploy to cloud instances using Ansible
  remote_start      Remotely start the metagraph on cloud instances using Ansible
```
TIP: You can use the same `-h` in each command listed above to see the accepted parameters

### Building
Let's start with the `build` command. This command could be used simply this way:
```
./hydra build   
```
This script has some parameters such as `--no_cache` (run without previous cache), `--run` (automatically run after build), `--only` (to build a specifical container), and `--include_dag_l1` (include the dag-l1 layer).

If you provide the `--run` parameter you should see the available URLs at the end of script execution

### Starting
We have the options `start_genesis` and `start` to start the containers. This option will fail case you didn't build the containers yet. You can call the option this way:
```
./hydra start_genesis
./hydra start   
```

This script has some parameters such as `--only` (to start a specifical container), and `--include_dag_l1` (include the dag-l1 layer).

You should see the URLs at the end

### Stopping
We have the option `stop` to stop the containers. You can call the option this way:
```
./hydra stop   
```
This script has the parameter `--only` (to stop a specifical container).

### Destroying
We have the option `destroiy` to destroy the containers. You can call the option this way:
```
./hydra destroy   
```
This script has some parameters such as `--only` (to stop a specifical container), and `--delete_local_codebase` (delete your local codebases with containers).

### Status
We have the option `status` to show the containers status. You can call the option this way:
```
./hydra status   
```
This script has the parameter `--show_all` (to include stopped containers at the listing).

### Installing
We have the option `install` to remove the link with remote `git`. You can call the option this way:
```
./hydra install   
```
## Let's build

After understanding the folder structure, we can start build our containers.

*NOTE: Make sure to fill your GITHUB_TOKEN on euclid.json file before start*

Move your terminal to directory `/scripts`, home of the `hydra` script.

```
  cd scripts/
```

We need to install `argc` to run the script, [here](https://github.com/sigoden/argc) is the doc of `argc`

```
cargo install argc
```

Then run the following to build your containers
```
./hydra build
```

After the end of this step, run the following:
```
./hydra start_genesis
```

After the end of `start_genesis`, you should see something like this:
```
Containers successfully built. URLs:
Global L0: http://localhost:9000/cluster/info
Metagraph L0 - 1: http://localhost:9400/cluster/info
Metagraph L0 - 2: http://localhost:9500/cluster/info
Metagraph L0 - 3: http://localhost:9600/cluster/info
Metagraph L1 Currency - 1: http://localhost:9700/cluster/info
Metagraph L1 Currency - 2: http://localhost:9800/cluster/info
Metagraph L1 Currency - 3: http://localhost:9900/cluster/info
Metagraph L1 Data - 1: http://localhost:8000/cluster/info
Metagraph L1 Data - 2: http://localhost:8100/cluster/info
Metagraph L1 Data - 3: http://localhost:8200/cluster/info
Grafana: http://localhost:3000/

```
You can now access the URLs and see that your containers are working properly

You can also call the `hydra` option
```
./hydra status
```

## Monitoring
With the containers building/starting we also build a monitoring tool. You can access this tool at this URL: `http://localhost:3000/`. The initial login and password are:
```
username: admin
password: admin
```
You'll be requested to update the password after your first login

In this tool we have 2 dashboards, you can access them on `Dashboard` section


## Deployment

Configuring, deploying, and starting to remote node instances is supported through Ansible playbooks. The default settings deploy to three node instances via SSH which host all layers of your metagraph project (gL0, mL0, cL1, dL1). Two hydra methods are available to help with the deployment process: `hydra remote_deploy` and `hydra remote_start`.
Prior to running these methods, remote host information must be configured in  `infra/ansible/hosts.ansible.yml`

### `hydra remote_deploy`
This method configures remote instances with all the necessary dependencies to run a Metagraph, including Java, Scala, and required build tools. The Ansible playbook used for this process can be found and edited in `infra/ansible/playbooks/deploy.ansible.yml`. Also, creates all required directories on the remote hosts, and creates or updates metagraph files to match your local Euclid environment. Specifically, it creates the following directories:

-   `code/global-l0`
-   `code/metagraph-l0`
-   `code/currency-l1`
-   `code/data-l1`

Each directory will be created with `cl-keytool.jar`, `cl-wallet.jar`, and a P12 file for the instance. Additionally, they contain the following:

**In `code/metagraph-l0`:**
-   metagraph-l0.jar     // The executable for the mL0 layer
-   genesis.csv              // The initial token balance allocations
-   genesis.snapshot    // The genesis snapshot created locally
-   genesis.address      // The metagraph address created in the genesis snapshot
-   
**In `code/currency-l1`:**
-   currency-l1.jar     // The executable for the cL1 layer
-   
**In `code/data-l1`:**
-   data-l1.jar     // The executable for the dL1 layer


### `hydra remote_start`

This method initiates the remote startup of your metagraph in one of the available networks: testnet, integrationnet, and mainnet.

To commence the remote startup of the metagraph, we utilize the parameters configured on Euclid. The startup process unfolds as follows:

1.  Termination of any processes currently running on the metagraph ports, which by default are 7000 for ml0, 8000 for cl1, and 9000 for dl1.
2.  Relocation of any existing logs to a folder named `older-logs`, residing within each layer directory: `metagraph-l0`, `currency-l1`, and `data-l1`.
3.  Initiation of the `metagraph-l0` layer, with `node-1` designated as the genesis node.
4.  Initial startup as `genesis`, transitioning to `rollback` for subsequent executions. To force a genesis startup, utilize the `--force_genesis` flag with the `hydra remote_start` command.  This will move the current `data` directory to a folder named `older-datas`.
5.  Detection of missing files required for layer execution, such as `:your_file.p12` and `metagraph-l0.jar`, triggering an error and halting execution.
6.  Following the initiation of `metagraph-l0`, the l1 layers, namely `currency-l1` and `data-l1`, are started. These layers only commence if their respective JARs are present in the directories. Unlike `metagraph-l0`, the absence of JARs does not prompt an error but rather prevents startup.

After the script completes execution, you can verify if your metagraph is generating snapshots by checking the block explorer of the selected network:

-   Testnet: [https://be-testnet.constellationnetwork.io/currency/:your_metagraph_id/snapshots/latest](https://be-testnet.constellationnetwork.io/currency/:your_metagraph_id/snapshots/latest)
-   Integrationnet: [https://be-integrationnet.constellationnetwork.io/currency/:your_metagraph_id/snapshots/latest](https://be-integrationnet.constellationnetwork.io/currency/:your_metagraph_id/snapshots/latest)
-   Mainnet: [https://be-mainnet.constellationnetwork.io/currency/:your_metagraph_id/snapshots/latest](https://be-mainnet.constellationnetwork.io/currency/:your_metagraph_id/snapshots/latest)

**NOTE:** Ensure that your peerIDs are registered on the desired network; otherwise, the metagraph startup will fail because the network will reject the snapshots.
**NOTE:** Don't forget to add your hosts' information, such as host, user, and SSH key file, to your `infra/ansible/hosts.ansible.yml` file.