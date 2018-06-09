# Initial Cluster Setup
For definitions of terminology used, please refer to our [terminology reference](https://github.com/hysds/hysds-framework/wiki/Docs#general-hysds-terminology).

When installation of the HySDS framework is complete on your `mozart` instance (see [[Installation]]), we must configure the rest of the cluster instances so that they can talk to each other. We do this using the `sdscli` command on the `mozart` instance. The idea is that all code and configuration is centralized on the `mozart` instance and when ready to deploy updates during the development cycle or when upgrading operations, we can push them easily from a single location.

1. Configure your cluster parameters using `sdscli`:
   The `sdscli` repo was installed on you `mozart` instance during [[Installation]]. Configure your cluster by running:
   ```
   cd ~
   source ~/mozart/bin/activate
   sds configure
   ```
1. The `sds configure` command will prompt you for your cluster parameters. A description of the parameters with examples is provided below

*** LOOK OVER THE FIELDS AND HAVE THE VALUES READY BEFORE HAND ***

   | field | description | example |
   | ----- | ----------- | ------- |
   | MOZART_PVT_IP | private IP address of `mozart` instance | 100.64.134.201 |
   | MOZART_PUB_IP | publicly accessible IP address of `mozart` instance, e.g. Elastic IP; can be the same as MOZART_PVT_IP | 64.34.21.123 |
   | MOZART_FQDN | publicly resolvable FQDN of `mozart` instance; can be the same as MOZART_PVT_IP | gman-jobs.hysds.net |
   | MOZART_RABBIT_PVT_IP | private IP address of `mozart` rabbitMQ instance (if running rabbitMQ on a different instance); otherwise replicate value from MOZART_PVT_IP | 100.64.134.201 |
   | MOZART_RABBIT_PUB_IP | publicly accessible IP address of `mozart` rabbitMQ instance (if running rabbitMQ on a different instance); otherwise replicate value from MOZART_PUB_IP | 64.34.21.123 |
   | MOZART_RABBIT_FQDN | publicly resolvable FQDN of `mozart` rabbitMQ instance (if running rabbitMQ on a different instance); otherwise replicate value from MOZART_FQDN | gman-jobs.hysds.net |
   | MOZART_RABBIT_USER | rabbitMQ user account; default installed by rabbitMQ is `guest` | guest |
   | MOZART_RABBIT_PASSWORD | rabbitMQ user password; default installed by rabbitMQ is `guest` | guest |
   | MOZART_REDIS_PVT_IP | private IP address of `mozart` redis instance (if running redis on a different instance); otherwise replicate value from MOZART_PVT_IP | 100.64.134.201 |
   | MOZART_REDIS_PUB_IP | publicly accessible IP address of `mozart` redis instance (if running redis on a different instance); otherwise replicate value from MOZART_PUB_IP | 64.34.21.123 |
   | MOZART_REDIS_FQDN | publicly resolvable FQDN of `mozart` redis instance (if running redis on a different instance); otherwise replicate value from MOZART_FQDN | gman-jobs.hysds.net |
   | MOZART_ES_PVT_IP | private IP address of `mozart` elasticsearch instance (if running elasticsearch on a different instance); otherwise replicate value from MOZART_PVT_IP | 100.64.134.201 |
   | MOZART_ES_PUB_IP | publicly accessible IP address of `mozart` elasticsearch instance (if running elasticsearch on a different instance); otherwise replicate value from MOZART_PUB_IP | 64.34.21.123 |
   | MOZART_ES_FQDN | publicly resolvable FQDN of `mozart` elasticsearch instance (if running elasticsearch on a different instance); otherwise replicate value from MOZART_FQDN | gman-jobs.hysds.net |
   | OPS_USER | ops account on HySDS cluster instances | ops or hysdsops or swotops |
   | OPS_HOME | ops account home directory on HySDS cluster instances | /home/ops or /data/home/hysdsops |
   | OPS_PASSWORD_HASH | sha224sum password hash for ops user account login to HySDS web interfaces | output of `echo -n mypassword \| sha224sum` |
   | LDAP_GROUPS | comma-separated list of LDAP groups to use for user authentication into HySDS web interfaces | hysds-v2,aria.users,ariamh |
   | KEY_FILENAME | private ssh key to use for logging into other cluster instances; used for deployment via fabric | /home/ops/.ssh/my_cloud_keypair.pem |
   | JENKINS_USER | account on `ci` instance that owns and runs the Jenkins CI server | jenkins |
   | JENKINS_DIR | location of the Jenkins HOME directory (where jobs/ directory is located) | /var/lib/jenkins |
   | METRICS_PVT_IP | private IP address of `metrics` instance | 100.64.134.153 |
   | METRICS_PUB_IP | publicly accessible IP address of `metrics` instance, e.g. Elastic IP; can be the same as METRICS_PVT_IP | 64.34.21.124 |
   | METRICS_FQDN | publicly resolvable FQDN of `metrics` instance; can be the same as METRICS_PVT_IP | gman-metrics.hysds.net |
   | METRICS_REDIS_PVT_IP | private IP address of `metrics` redis instance (if running redis on a different instance); otherwise replicate value from METRICS_PVT_IP | 100.64.134.153 |
   | METRICS_REDIS_PUB_IP | publicly accessible IP address of `metrics` redis instance (if running redis on a different instance); otherwise replicate value from METRICS_PUB_IP | 64.34.21.123 |
   | METRICS_REDIS_FQDN | publicly resolvable FQDN of `metrics` redis instance (if running redis on a different instance); otherwise replicate value from METRICS_FQDN | gman-metrics.hysds.net |
   | METRICS_ES_PVT_IP | private IP address of `metrics` elasticsearch instance (if running elasticsearch on a different instance); otherwise replicate value from METRICS_PVT_IP | 100.64.134.153 |
   | METRICS_ES_PUB_IP | publicly accessible IP address of `metrics` elasticsearch instance (if running elasticsearch on a different instance); otherwise replicate value from METRICS_PUB_IP | 64.34.21.124 |
   | METRICS_ES_FQDN | publicly resolvable FQDN of `metrics` elasticsearch instance (if running elasticsearch on a different instance); otherwise replicate value from METRICS_FQDN | gman-metrics.hysds.net |
   | GRQ_PVT_IP | private IP address of `grq` instance | 100.64.134.71 |
   | GRQ_PUB_IP | publicly accessible IP address of `grq` instance, e.g. Elastic IP; can be the same as GRQ_PVT_IP | 64.34.21.125 |
   | GRQ_FQDN | publicly resolvable FQDN of `grq` instance; can be the same as GRQ_PVT_IP | gman-grq.hysds.net |
   | GRQ_PORT | port to use for the grq2 REST API | 8878 |
   | GRQ_ES_PVT_IP | private IP address of `grq` elasticsearch instance (if running elasticsearch on a different instance); otherwise replicate value from GRQ_PVT_IP | 100.64.134.71 |
   | GRQ_ES_PUB_IP | publicly accessible IP address of `grq` elasticsearch instance (if running elasticsearch on a different instance); otherwise replicate value from GRQ_PUB_IP | 64.34.21.125 |
   | GRQ_ES_FQDN | publicly resolvable FQDN of `grq` elasticsearch instance (if running elasticsearch on a different instance); otherwise replicate value from GRQ_FQDN | gman-grq.hysds.net |
   | FACTOTUM_PVT_IP | private IP address of `factotum` instance | 100.64.134.184 |
   | FACTOTUM_PUB_IP | publicly accessible IP address of `factotum` instance, e.g. Elastic IP; can be the same as FACTOTUM_PVT_IP | 64.34.21.126 |
   | FACTOTUM_FQDN | publicly resolvable FQDN of `factotum` instance; can be the same as FACTOTUM_PVT_IP | gman-factotum.hysds.net |
   | CI_PVT_IP | private IP address of `ci` instance | 100.64.134.179 |
   | CI_PUB_IP | publicly accessible IP address of `ci` instance, e.g. Elastic IP; can be the same as CI_PVT_IP | 64.34.21.127 |
   | CI_FQDN | publicly resolvable FQDN of `ci` instance; can be the same as CI_PVT_IP | gman-ci.hysds.net |
   | VERDI_PVT_IP | private IP address of `verdi` instance; if no `verdi` instance, use `ci` instance value for CI_PVT_IP | 100.64.134.179 |
   | VERDI_PUB_IP | publicly accessible IP address of `verdi` instance, e.g. Elastic IP; if no `verdi` instance, use `ci` instance value for CI_PUB_IP | 64.34.21.127 |
   | VERDI_FQDN | publicly resolvable FQDN of `verdi` instance; if no `verdi` instance, use `ci` instance value for CI_FQDN | gman-ci.hysds.net |
   | JENKINS_API_USER | Jenkins user account to use for access to Jenkins API | gmanipon |
   | JENKINS_API_KEY | Jenkins user API key to use for access to Jenkins API. Go to an already set up Jenkins web page and click on “People”, your username, then “Configure”. Click on “Show API Token”. Use that token and you username for API_USER. | \<api key\> |
   | DAV_SERVER | WebDAV server for dataset publication (optional); leave blank if using S3 | aria-dav.jpl.nasa.gov |
   | DAV_USER | WebDAV server account with R/W access | ops |
   | DAV_PASSWORD | DAV_USER account password | \<password\> |
   | DATASET_AWS_ACCESS_KEY | AWS access key for account or role with R/W access to S3 bucket for dataset repository | \<access key\> |
   | DATASET_AWS_SECRET_KEY | AWS secret key for DATASET_AWS_ACCESS_KEY | \<secret key\> |
   | DATASET_AWS_REGION | AWS region for S3 bucket for dataset repository | us-west-2 |
   | DATASET_S3_ENDPOINT | S3 endpoint for the DATASET_AWS_REGION | s3-us-west-2.amazonaws.com |
   | DATASET_S3_WEBSITE_ENDPOINT | S3 website endpoint for the DATASET_AWS_REGION | s3-website-us-west-2.amazonaws.com |
   | DATASET_BUCKET | bucket name for dataset repository | ops-product-bucket |
   | AWS_ACCESS_KEY | AWS access key for account or role with R/W access to S3 bucket for code/config bundle and docker image repository; can be the same as DATASET_AWS_ACCESS_KEY | \<access key\> |
   | AWS_SECRET_KEY | AWS secret key for AWS_ACCESS_KEY; can be the same as DATASET_AWS_SECRET_KEY | \<secret key\> |
   | AWS_REGION | AWS region for S3 bucket for code/config bundle and docker image repository | us-west-2 |
   | S3_ENDPOINT | S3 endpoint for the AWS_REGION | s3-us-west-2.amazonaws.com |
   | CODE_BUCKET | bucket name for code/config bundle and docker image repository | ops-code-bucket |
   | VERDI_PRIMER_IMAGE | S3 url to `verdi` docker image in CODE_BUCKET | s3://ops-code-bucket/hysds-verdi-latest.tar.gz |
   | VERDI_TAG | docker tag for `verdi` docker image | latest |
   | VERDI_UID | UID of ops user on `ci` instance; used to sync UID upon docker image creation | 1001 |
   | VERDI_GID | GID of ops user on `ci` instance; used to sync GID upon docker image creation | 1001 |
   | AUTOSCALE_GROUP | prefix to use for naming autoscaling groups | e.g. my-test-autoscaling-group |
   | PROJECTS | space-delimited list of projects to create code/config bundles for | "dumby dumby_urgent" |
   | VENUE | unique tag name to differentiate this HySDS cluster from others | e.g. ops or dev or oasis or test |
   | PROVES_URL | url to PROV-ES server (optional) | https://prov-es.jpl.nasa.gov/beta |
   | PROVES_IMPORT_URL | PROV-ES API url for import of PROV-ES documents (optional) | https://prov-es.jpl.nasa.gov/beta/api/v0.1/prov_es/import/json |
   | DATASETS_CFG | location of [datasets configuration](https://github.com/hysds/hysds-framework/wiki/Datasets#dataset-configuration-how-to-configure-hysds-to-recognize-a-dataset-for-ingest) on workers | /home/ops/verdi/etc/datasets.json |
   | SYSTEM_JOBS_QUEUE | name of queue to use for system jobs | system-jobs-queue |
   | GIT_OAUTH_TOKEN | optional Github OAuth token to use on `ci` instance when checking out code for continuous integration (optional) | \<token\> |
1. Make sure elasticsearch is up on the mozart and grq instances. You can run the following command to check:
   ```
   curl 'http://<mozart/grq ip>:9200/?pretty'
   ```
   you should get answer back from ES, something like this:
   ```
   {
    "status" : 200,
    "name" : "Dweller-in-Darkness",
    "cluster_name" : "resource_cluster",
    "version" : {
      "number" : "1.7.3",
      "build_hash" : "05d4530971ef0ea46d0f4fa6ee64dbc8df659682",
      "build_timestamp" : "2015-10-15T09:14:17Z",
      "build_snapshot" : false,
      "lucene_version" : "4.10.4"
    },
    "tagline" : "You Know, for Search"
   }

   ```

   If you can not connect to elastic search, you need to start ElasticSearch in mozart and grq instances:

   ```
   sudo systemctl start elasticsearch
   ``` 
1. Ensure `mozart` component can connect to other components over ssh using the configured `KEY_FILENAME`. If correctly configured, the `sds status all` command should show that it was able to ssh into each component to check that the `supervisord` daemon was not running like below:
    ```
    sds status all
    ########################################
    grq
    ########################################
    [100.64.106.214] Executing task 'status'
    Supervisord is not running on grq.
    ########################################
    mozart
    ########################################
    [100.64.106.38] Executing task 'status'
    Supervisord is not running on mozart.
    ########################################
    metrics
    ########################################
    [100.64.106.140] Executing task 'status'
    Supervisord is not running on metrics.
    ########################################
    factotum
    ########################################
    [100.64.106.64] Executing task 'status'
    Supervisord is not running on factotum.
    ########################################
    ci
    ########################################
    [100.64.106.220] Executing task 'status'
    Supervisord is not running on ci.
    ########################################
    verdi
    ########################################
    [100.64.106.220] Executing task 'status'
    Supervisord is not running on verdi.
    ```
    Otherwise if any of the components show the following error, for example for the grq component:
    ```
    ########################################
    grq
    ########################################
    [100.64.106.214] Executing task 'status'

    Fatal error: Needed to prompt for a connection or sudo password (host: 100.64.106.214), but abort-on-prompts was set to True

    Aborting.
    Needed to prompt for a connection or sudo password (host: 100.64.106.214), but abort-on-prompts was set to True

    Fatal error: One or more hosts failed while executing task 'status'

    Aborting.
    One or more hosts failed while executing task 'status'
    ```
    then there is an issue with the configured `KEY_FILENAME` on `mozart` or the `authorized_keys` file under the component's `~/.ssh` directory for user `OPS_USER`. Resolve this issue before continuing on.
1. Update all HySDS components:
    ```
    sds update all
    ```
    If you receive any errors, they will need to be addressed.
1. Start up all HySDS components:
    ```
    sds start all
    ```
1. View status of HySDS components and services:
    ```
    sds status all
    ```
1. During installation, the latest versions of the `lightweight-jobs` core HySDS package and the `verdi` docker image was downloaded. Next we import the `lightweight-jobs` package:
    ```
    cd ~/mozart/pkgs
    sds pkg import container-hysds_lightweight-jobs.*.sdspkg.tar
    ```
1. Finally we copy the `verdi` docker image to the code bucket (`CODE_BUCKET` as specified during `sds configure`). Ensure `VERDI_PRIMER_IMAGE` url is consistent:
    ```
    aws s3 cp hysds-verdi-latest.tar.gz s3://<CODE_BUCKET>/hysds-verdi-latest.tar.gz
    ```

### Next Steps
Now that you have your HySDS cluster configured, continue on to **[[Hello World]].**

To configure Autoscaling groups for your HySDS cluster, continue on to **[[Create-AWS-Autoscaling-Group-for-Verdi]]**.

To configure a staging area for your HySDS cluster, continue on to **[[Create-AWS-Resources-for-Staging-Area]]**.
