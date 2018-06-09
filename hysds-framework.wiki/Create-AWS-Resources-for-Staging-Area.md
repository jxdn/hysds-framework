# Setting up an AWS Resources for Staging Area
For definitions of terminology used, please refer to our [terminology reference](https://github.com/hysds/hysds-framework/wiki/Docs#general-hysds-terminology).

In this tutorial, we use the `sdscli` to create a staging area in your dataset bucket. The staging area will be a prefix ("directory") at the top level of your dataset bucket where files/directories can be deposited to trigger arbitrary HySDS jobs. As an example, assume in our `.sds/config` we've set `DATASET_BUCKET` to `gman-swot-dataset-bucket`. By default, the `sdscli cloud storage create_staging_area` command uses `staging_area` as the S3 key prefix to use but that can be specified using the `--prefix` option. A file can be deposited to `s3://gman-swot-dataset-bucket/staging_area/`, for example:

   ```
   aws s3 cp LC08_L1TP_149039_20170411_20170415_01_T1_B2.TIF s3://gman-swot-dataset-bucket/staging_area/
   ```

Other files or directories can be staged but nothing is triggered until the signal file is deposited. By default the `sdscli cloud storage create_staging_area` command uses `.met.json` as the S3 key suffix to use but that can be specified using the `--suffix` option. The contents of the signal file should ccontain information that will feed into the arbitrary HySDS job to be triggered and the filename format of the signal should be the file or directory that was staged with the suffix at the end. Continuing our example, we deposit the signal file:

   ```
   echo '{ "id": "LC08_L1TP_149039_20170411_20170415_01_T1_B2", "geolocation": [ 1, 2, 3 ,4 ], "whatever": "okay" }' > LC08_L1TP_149039_20170411_20170415_01_T1_B2.TIF.met.json
   aws s3 cp LC08_L1TP_149039_20170411_20170415_01_T1_B2.TIF.met.json s3://gman-swot-dataset-bucket/staging_area/
   ```

Upon the `s3:ObjectCreated:*` event of the `s3://gman-swot-dataset-bucket/staging_area/LC08_L1TP_149039_20170411_20170415_01_T1_B2.TIF.met.json`, an S3 event notification is sent to a pre-configured SNS topic. The subscriber of the SNS topic is a pre-configured Lambda function that will read in the contents of the S3 event payload (which includes the S3 path of the signal file that was deposited) and the contents of the signal file and submit them to an arbitrary HySDS job. This job is arbirtrary but usually in an SDS the job should extract metadata from the staged data, create a formal HySDS dataset, ingest it and finally clean it out from the staging area.

We use the `sdscli` to create all the AWS resources needed to setup our staging area. You'll need the following information beforehand to facilitate provisioning:

   - AWS security groups to use for lambda execution
   - AWS role to use for lambda execution
   - HySDS job type to submit data staged event to, e.g. INGEST_L0A_LR_RAW
   - release version of the HySDS job type, e.g. release-20180327
   - HySDS queue name to submit the job to, e.g. factotum-job_worker-small


We assume that you have an AWS account, have configured your HySDS cluster in EC2 and have completed cluster setup per **[[Cluster Setup]]**.

1. Log into `mozart` instance and source the mozart virtual environment:
   ```
   source ~/mozart/bin/activate
   ```
1. Run `sdscli` command to create staging area:
   ```
   sds cloud storage create_staging_area
   ```

   The command will prompt you for information needed to create all of the resources:
   ```
   Current security groups are:
   
   0. vpc-12345678 - PCM Factotum and CI - sg-12345678
   1. vpc-87654321 - default - sg-87654321
   
   Select security groups lambda will use (space between each selected): 1
   Current roles are:
   
   0. arn:aws:iam::123456789101:role/service-role/config-role-us - arn:aws:iam::123456789101:role/service-role/config-role-us (2016-11-15 22:25:08+00:00)
   1. arn:aws:iam::123456789101:role/pcm-lambda-role - arn:aws:iam::123456789101:role/pcm-lambda-role (2017-12-12 21:22:22+00:00)
   2. arn:aws:iam::123456789101:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling - arn:aws:iam::123456789101:role/aws-service-ro
   
   Select role to use for lambda execution: 1
   Enter job type to submit on data staged event: INGEST_L0A_LR_RAW
   Enter release version for INGEST_L0A_LR_RAW: release-20180327
   Enter queue name to submit INGEST_L0A_LR_RAW-master jobs to: factotum-job_worker-small
   ```
1. Navigate to your AWS dashboard and check that the S3 event on your dataset bucket was created:

   [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/sdscli-s3_event.png|alt=s3_event]]

1. Check that the SNS topic was created:

   [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/sdscli-sns_topic.png|alt=sns_topic]]

1. Finally check that the lambda function was created:

   [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/sdscli-data_staged_lambda.png|alt=data_staged_lambda]]
   

### That's it!
Congratulations, you've created a staging area for your HySDS cluster.
