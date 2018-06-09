# Setting up an AWS Autoscaling Group for Verdi (worker) fleet
For definitions of terminology used, please refer to our [terminology reference](https://github.com/hysds/hysds-framework/wiki/Docs#general-hysds-terminology).

## Prerequisites
- You should have already created an `autoscale` AMI. If it is not provisioned to you by an administrator, you can provision them yourself using our puppet module:
  - `autoscale` - https://github.com/hysds/puppet-autoscale

## Create `verdi` image using the `autoscale` AMI
The autoscale puppet module creates a base `verdi` AMI that is provisioned with bootstrap scripts and cloud utilities to automatically configure itself to your HySDS cluster as an autoscaling `verdi` worker. In order for the instance to bootstrap itself, it needs to be populated with your cluster's AWS credentials in order to pull the code/configuration bundle from S3.

1. Log into the AWS dashboard and on the EC2 page, select "AMIs" on the left panel. You should see your `autoscale` AMI listed there if the prerequisites were completed. Click on the checkbox next to it to select it and click "Launch".
1. Select the `c5.large` instance type and click "Next: Configure Instance Details".
1. Select the proper VPC under "Network" and click on "Next: Add Storage".
1. Ensure that there are 3 volumes: the "Root" volume should already be there. If need be, add 2 new EBS volumes. For the first EBS volume, select "/dev/sdb" and set the size to 50GB. Click on the "Delete on Termination" checkbox to ensure that it will be deleted when the insetance is terminated. For the second EBS volume, select "/dev/sdc" and set the size to 100GB. Also make sure that "Deleted on Termination" is checked on for this EBS volume. Finally click on "Next: Add Tags".
1. Skip adding tags and click on "Next: Configure Security Groups".
1. Select the "default" security group and all other security groups needed to allow instances to communicate with your HySDS cluster. click on "Review and Launch".
1. Double check to make sure all parameters are correct and click on "Launch" to create your `autoscale` instance.
1. Select the keypair that was used when launching your HySDS cluster instances during [[Installation]]. Click on the acknowledgement and finally the "Launch Instances" button.
1. Once instance is running, ssh into the instance and configure the AWS credentials of the `ops` user (run `aws configure`). Test that the credential works and can list the buckets in your account (e.g. run `aws s3 ls`).
1. In the AWS dashboard, select this instance, click on the "Actions" button and navigate to the "Image" submenu and select "Create Image". Name the image "CentOS 7 PROJECT Verdi YYYY-MM-DD" and replace `PROJECT` with anything you desire to differentiate this AMI with other `verdi` AMIs. Also replace `YYYY-MM-DD` with today's date. Optionally add an image description. 
1. Ensure that the "Delete on Termination" checkboxes are checked on all 3 volumes/devices: /dev/sda1, /dev/sdb, /dev/sdc. Finally click on "Create Image".
1. Once the AMI has been created successfully, terminate the instance.

## Create AWS Autoscaling group using the `sdscli`
In this tutorial, we use the `sdscli` to create Autoscaling Group (ASG) for our cluster to automatically scale out the number of worker instances based on a custom target tracking metric, i.e. JobsWaitingPerInstance. We assume that you have an AWS account, have configured your HySDS cluster in EC2 and have completed cluster setup per **[[Cluster Setup]]**.

1. Log into `mozart` instance and source the mozart virtual environment:
   ```
   source ~/mozart/bin/activate
   ```
1. Run `sdscli` command to create cloud autoscaling groups for your cluster:
   ```
   sds cloud asg create   
   ```

   The command will prompt you for information needed to create the launch configurations and autoscaling groups:
   ```
   Current verdi AMIs are:
   
   0. SWOT PCM VERDI 2017-09-26 - ami-12345678 (2017-09-26T20:28:48.000Z)
   1. CentOS 7 SWOT Verdi 2017-10-04 - ami-12345679 (2017-10-04T03:23:59.000Z)
   2. CentOS 7 SWOT Verdi 2017-12-11 - ami-12345677 (2017-12-11T22:56:59.000Z)
   3. CentOS 7 SWOT Verdi 2017-12-15 - ami-12345676 (2017-12-15T01:51:23.000Z)
   4. CentOS 7 SWOT Verdi 2018-02-14 - ami-12345675 (2018-02-14T19:28:26.000Z)
   
   Select verdi AMI to use for launch configurations: 4
   Current key pairs are:
   
   0. gman-test
   1. gman
   2. pcmdev
   
   Select key pair to use for launch configurations: 0
   Current security groups are:
   
   0. vpc-12345678 - PCM Factotum and CI - sg-12345678
   1. vpc-12345678 - default - sg-12345679
   2. vpc-12345678 - Verdi - sg-12345670
   
   Select security groups to use for launch configurations (space between each selected): 1 2
   ########################################
   Configuring autoscaling group:
   gman-swot-asg-r1-swot-dev
   ########################################
   Refer to https://www.ec2instances.info/ and enter instance type to use for launch configuration: c3.xlarge
   Do you want to use spot instances [y/n]: y
   Enter spot price bid: 0.21
   Created launch configuration gman-swot-asg-r1-swot-dev-c3.xlarge-spot-0.21-launch-config.
   Created autoscaling group gman-swot-asg-r1-swot-dev
   Added target tracking scaling policy gman-swot-asg-r1-swot-dev-large-target-tracking to gman-swot-asg-r1-swot-dev
   Added target tracking scaling policy gman-swot-asg-r1-swot-dev-small-target-tracking to gman-swot-asg-r1-swot-dev
   ########################################
   Configuring autoscaling group:
   gman-swot-asg-r1-swot_urgent-dev
   ########################################
   Refer to https://www.ec2instances.info/ and enter instance type to use for launch configuration: c3.xlarge
   Do you want to use spot instances [y/n]: y
   Enter spot price bid: 0.21
   Created launch configuration gman-swot-asg-r1-swot_urgent-dev-c3.xlarge-spot-0.21-launch-config.
   Created autoscaling group gman-swot-asg-r1-swot_urgent-dev
   Added target tracking scaling policy gman-swot-asg-r1-swot_urgent-dev-large-target-tracking to gman-swot-asg-r1-swot_urgent-dev
   Added target tracking scaling policy gman-swot-asg-r1-swot_urgent-dev-small-target-tracking to gman-swot-asg-r1-swot_urgent-dev
   ```
1. Navigate to your AWS dashboard and check that the Autoscaling groups and launch configurations were created fine:

   [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/sdscli-autoscaling_groups.png|alt=autoscaling]]

   [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/sdscli-launch_configs.png|alt=launch_configs]]

1. Note that the min/max/desired values for the Autoscaling groups are set to 0 by default. Modify them accordingly to allow scale up.
   

### That's it!
Congratulations, you've created Autoscaling groups for your HySDS cluster's worker fleets.
