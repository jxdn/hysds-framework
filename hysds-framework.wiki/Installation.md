# Installation
this wiki is modified from : https://github.com/hysds/  (NASA JPL)

For definitions of terminology used, please refer to our [terminology reference](https://github.com/hysds/hysds-framework/wiki/Docs#general-hysds-terminology).

## Prerequisites
- You should already have provisioned 5 instances each designated for a particular HySDS component. The suggested `minimum` CPU and memory requirements for each instance should at least be met (tables below). If they are not provisioned to you by an administrator, you can provision them yourself using our puppet modules:
  1. `mozart` - https://bitbucket.org/nvgdevteam/puppet-mozart
  1. `metrics` - https://bitbucket.org/nvgdevteam/puppet-metrics
  1. `grq` - https://bitbucket.org/nvgdevteam/puppet-grq
  1. `factotum` - https://bitbucket.org/nvgdevteam/puppet-factotum
  1. `ci` - https://bitbucket.org/nvgdevteam/puppet-cont_int
- The following tables list the CPU and memory requirements for each component instance based on the scope of the HySDS cluster: `minimum`, `development`, `operations`:
  - `minimum` - Uses instance types that cover the bare minimum CPU and memory requirements for running HySDS in a test environment such as this user guide, e.g. [[Hello World]] and [[Hello Dataset]]

   | component | CPU | memory (GB) | instance type |
   | --------- | --- | ----------- | ------------- |
   | mozart | 2 | 7 | AWS: `m3.large`; GCP: `n1-standard-2`; Azure: `D2 v3` |
   | metrics | 2 | 7 | AWS: `m3.large`; GCP: `n1-standard-2`; Azure: `D2 v3` |
   | grq | 2 | 7 | AWS: `m3.large`; GCP: `n1-standard-2`; Azure: `D2 v3` |
   | factotum | 2 | 7 | AWS: `m3.large`; GCP: `n1-standard-2`; Azure: `D2 v3` |
   | ci | 2 | 7 | AWS: `m3.large`; GCP: `n1-standard-2`; Azure: `D2 v3` |
  - `development` - Uses instance types that cover the CPU and memory requirements for running HySDS in a development environment

   | component | CPU | memory (GB) | instance type |
   | --------- | --- | ----------- | ------------- |
   | mozart | 4 | 30 | AWS: `r4.xlarge`; GCP: `n1-standard-8`; Azure: `E4 v3` |
   | metrics | 4 | 30 | AWS: `r4.xlarge`; GCP: `n1-standard-8`; Azure: `E4 v3` |
   | grq | 4 | 30 | AWS: `r4.xlarge`; GCP: `n1-standard-8`; Azure: `E4 v3` |
   | factotum | 4 | 15 | AWS: `m3.xlarge`; GCP: `n1-standard-4`; Azure: `B4MS` |
   | ci | 4 | 15 | AWS: `m3.xlarge`; GCP: `n1-standard-4`; Azure: `B4MS` |
  - `optimal` - Uses instance types that cover the CPU and memory requirements for running HySDS in an operational environment or at large scale (500-8000 `verdi` worker instances)

   | component | CPU | memory (GB) | instance type |
   | --------- | --- | ----------- | ------------- |
   | mozart | 32 | 208 | AWS: `r4.8xlarge`; GCP: `n1-highmem-32`; Azure: `E32 v3` |
   | metrics | 32 | 208 | AWS: `r4.8xlarge`; GCP: `n1-highmem-32`; Azure: `E32 v3` |
   | grq | 32 | 208 | AWS: `r4.8xlarge`; GCP: `n1-highmem-32`; Azure: `E32 v3` |
   | factotum | 32 | 60 | AWS: `c3.8xlarge`; GCP: `n1-standard-32`; Azure: `F32 v2` |
   | ci | 32 | 60 | AWS: `c3.8xlarge`; GCP: `n1-standard-32`; Azure: `F32 v2` |
- Ensure that the same SSH keypair is used for all 5 of the instances. You should be able to ssh into any of your instances using the PEM key file of this SSH keypair.
- You should already have provisioned 2 buckets from a cloud vendor (e.g. AWS S3, Azure Blob, Google Cloud Storage) 
  - one for code and configuration (`CODE_BUCKET`), e.g. s3://my-code-bucket
  - one for datasets and staging area (`DATASET_BUCKET`), e.g. s3://my-dataset-bucket

## Install HySDS framework on each instance
The latest releases are here: https://bitbucket.org/nvgdevteam/hysds-framework/releases.
Each was taken from the latest head state of all repos at time of release.

### Install HySDS framework on mozart
1. Log into your `mozart` instance
   ```
   ssh -i <PEM file> ops@<mozart IP>
   ```
1. Copy the PEM file that you used to log into the `mozart` instance into ~/.ssh directory. You can use scp to transfer the file:
   ```
   scp -i <PEM file> <PEM file> ops@<mozart IP>:~/.ssh
   ```
1. Ensure the permissions on the PEM file are readable by your account only:
   ```
   chmod 400 <PEM file>
   ```
1. If there's a `mozart` directory already under you home directory, move or remove it:
   ```
   mv ~/mozart ~/mozart.orig

   or 

   rm -rf ~/mozart
   ```
1. Clone the HySDS framework repository and enter it
   ```
   cd $HOME
   git clone https://bitbucket.org/nvgdevteam/hysds-framework.git
   cd hysds-framework
   ```
1. Select the HySDS framework release tag you'd like to install for mozart
   ```
   $ ./install.sh mozart
   HySDS install directory set to /home/ops/mozart
   New python executable in /home/ops/mozart/bin/python
   Installing Setuptools............................................done.
   Installing Pip...................................................done.
   Created virtualenv at /home/ops/mozart.
   [2017-08-09 19:25:37,789: INFO/main] Github repo URL: https://xxxxxxxx@github.com/api/v3/repos/hysds/hysds-framework/releases
   [2017-08-09 19:25:37,798: INFO/_new_conn] Starting new HTTPS connection (1): github.com
   No release specified. Use -r RELEASE | --release=RELEASE to install a specific release. Listing available releases:
   v2.0.0-beta.3
   v2.1.0-beta.0
   v2.1.0-beta.1
   v2.1.0-beta.2
   v2.1.0-beta.3
   v2.1.0-beta.4
   ```
1. Install the HySDS release you want to install (e.g. v2.1.0-beta.1) for the mozart component
   ```
   $ ./install.sh mozart -r <release>

   e.g.
   
   $ ./install.sh mozart -r v2.1.0-beta.4
   ```
   You could also install the development version which pulls the master branch of each HySDS repo:
   ```
   $ ./install.sh mozart -d
   ```
### Next Steps
Now that you have the HySDS framework installed on your cluster's mozart instance, continue on to **[[Cluster Setup]].**
