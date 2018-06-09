# Integrating a "Hello World" Job Type
For definitions of terminology used, please refer to our [terminology reference](https://github.com/hysds/hysds-framework/wiki/Docs#general-hysds-terminology).

When installation (see [[Installation]]) and setup of your HySDS cluster (see [[Cluster Setup]]) is complete, you can start integrating job types (PGE or non-PGE) into the system. In this tutorial, we integrate our first job type which will simply echo "Hello World".

1. On the `mozart` instance, create a directory for your new job type
   ```
   cd ~/mozart/ops
   mkdir hello_world
   cd hello_world
   ```
2. Let's create the script that runs our job type. Open up a new file
   ```
   vi run_hello_world.sh
   ```
   and paste the contents below
   ```
   #!/bin/bash
   echo "Hello World"
   echo "Hello World to STDERR" 1>&2
   ```
   and save.
3. Set the script to be executable
   ```
   chmod 755 run_hello_world.sh
   ```
4. Test the script by running it
   ```
   ./run_hello_world.sh  
   Hello World
   Hello World to STDERR
   ```
5. Now let's create the necessary files for integrating this job into your HySDS cluster. Create the `docker` directory and cd into it
   ```
   mkdir docker
   cd docker
   ```
6. Create a Dockerfile for your job type by opening up a new file `Dockerfile.<job_type_name>`
   ```
   vi Dockerfile.hello_world
   ```
   paste the following into it
   ```
   FROM hysds/pge-base

   ARG id
   ARG gid

   # copy job repo
   COPY . /home/ops/verdi/ops/hello_world
   RUN set -ex \
    && sudo chown -R ops:ops /home/ops/verdi/ops/hello_world

   # set entrypoint
   USER ops
   WORKDIR /home/ops
   CMD ["/bin/bash", "--login"]
   ```
   and save.
7. Create a `job-spec` file for your job type by opening up a new file `job-spec.json.<job_type_name>`
   ```
   vi job-spec.json.hello_world
   ```
   paste the following into it
   ```
   {
     "command": "/home/ops/verdi/ops/hello_world/run_hello_world.sh",
     "recommended-queues": [ "job_worker-small" ],
     "params": []
   }
   ```
   and save. For more information on the `job-spec` format, check out [Job and HySDS IO Specifications](https://github.com/hysds/hysds-framework/wiki/Job-and-HySDS-IO-Specifications).
8. Create a `hysds-io` file for your job type by opening up a new file `hysds-io.json.<job_type_name>`
   ```
   vi hysds-io.json.hello_world
   ```
   paste the following into it
   ```
   {
     "label": "Hello World",
     "params": []
   }
   ```
   and save. For more information on the `hysds-io` format, check out [Job and HySDS IO Specifications](https://github.com/hysds/hysds-framework/wiki/Job-and-HySDS-IO-Specifications).
9. Change directory back up to the root of your job type directory
   ```
   cd ..
   ```
   and the layout of the contents should be as follows
   ```
   |-- docker
   |   |-- Dockerfile.hello_world
   |   |-- hysds-io.json.hello_world
   |   `-- job-spec.json.hello_world
   `-- run_hello_world.sh

   1 directory, 4 files
   ```
10. Now we create a git repository from this directory
    ```
    git init
    git add .
    git status
    # On branch master
    #
    # Initial commit
    #
    # Changes to be committed:
    #   (use "git rm --cached <file>..." to unstage)
    #
    #       new file:   docker/Dockerfile.hello_world
    #       new file:   docker/hysds-io.json.hello_world
    #       new file:   docker/job-spec.json.hello_world
    #       new file:   run_hello_world.sh
    git commit -a -m "initial import"
    ```
11. Create a new repository on github, copy the repo URL, and push your existing hello_world repository. In this example, I created a new repo under my github user account, https://github.com/pymonger/hello_world.git
    ```
    git remote add origin https://github.com/<your github account>/hello_world.git
    git push -u origin master
    Counting objects: 7, done.
    Delta compression using up to 32 threads.
    Compressing objects: 100% (6/6), done.
    Writing objects: 100% (7/7), 794 bytes | 0 bytes/s, done.
    Total 7 (delta 0), reused 0 (delta 0)
    To https://github.com/<your github account>/hello_world.git
     * [new branch]      master -> master
    Branch master set up to track remote branch master from origin.
    ```
12. Ensure that your HySDS cluster is up and running
    ```
    sds status
    ```
    If it is not running, run `sds start all`.
13. Now we need to configure our new job type for continuous integration and deployment. Since we're still developing this job type, we'll configure it using the master branch instead of release tags
    ```
    sds ci add_job https://github.com/<your github account>/hello_world.git s3 $UID $(id -g) -b master
    ```
    If jenkins needs to use an OAuth token to access the repo, specify the `-k` option and ensure that `GIT_OAUTH_TOKEN` was specified in `$HOME/.sds/config`:
    ```
    sds ci add_job https://github.com/<your github account>/hello_world.git s3 $UID $(id -g) -b master -k
    ```
    You should have output similar to the following
    ```
    Jenkins job create for:

      https://github.com/pymonger/hello_world.git

    Please ensure that a webhook has been configured at:

      https://github.com/pymonger/hello_world/settings/hooks

    to push 'Push' and 'Repository' events to:

      http://gman-dv-ci.hysds.net:8080/github-webhook/

    If you've configured a Jenkins account with an OAuth credential and full 
    access to your github account, this may be done automatically for you.
    ```
    Follow these instructions to configure the webhook on github so that it can trigger rebuild and deployment.

    Note: If you run into issues like : 
    ```
    [192.168.183.14] run: java -jar /home/ops/jenkins/war/WEB-INF/jenkins-cli.jar -s http://localhost:8080 -http -auth mkarim:9080cc69e123055d68ee19ff0dfa098a reload-configuration
    [192.168.183.14] out: No such command: -http
    [192.168.183.14] out:
    ```
    you may have a jenkins-cli.jar version that does not support -http, -auth parameters. You can either get a latest jenkins-cli.jar, or can make the following change line in cluster.py to make it work:
    ```
    run('java -jar %s/war/WEB-INF/jenkins-cli.jar -s http://localhost:8080 -http -auth %s:%s reload-configuration' %(ctx['JENKINS_DIR'], juser,jkey))
    ```
    to:

    ```
    run('java -jar %s/war/WEB-INF/jenkins-cli.jar -s http://localhost:8080 reload-configuration' % ctx['JENKINS_DIR'])
    ```

    If the build fails with following error (aws encryption type) :
    ```
     out: aws: error: argument --sse: ignored explicit argument 'AES256'
    ``` 

    remove the encryption type value as AES256 is default type in cluster.py:

    ```
      run('aws s3 cp --sse=AES256 %s s3://%s/' % (tar_file, ctx['CODE_BUCKET']))
    ```
    to:
    ```
     run('aws s3 cp --sse %s s3://%s/' % (tar_file, ctx['CODE_BUCKET']))
    ```

    if your build fails with import error for python-pyasn1 or rsa:
    ```
    ImportError: cannot import name opentype
    [ERROR] Failed to make metadata and store container for: container-hello_world:master
    ```
    you may not have latest version of the packages. use pip to install them (specially in ci and factotum instances):

    ```
    sudo pip install pyasn1_modules
    sudo pip install rsa
    ```
    yum may not install the latest version of pyasn1_modules.


14. Navigate to your `ci` instance (e.g. http://\<your-ci-instance\>:8080) to validate that the jenkins job was configured

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-jenkins.png|alt=jenkins]]
    You should see your jenkins job named as "container-builder_\<your repo name\>_\<branch\>" (e.g. container-builder_gmanipon_hello_world_master).
15. Verify that the jenkins job runs to completion by manually scheduling a build. The jenkins job will build the docker image, publish it to your S3 code bucket, and register the job types into you HySDS cluster. Click on the green arrow and view the console output to validate.
16. Verify that your new job type was published to your HySDS cluster. Open up a browser and point it to your `mozart` instance's ElasticSearch head plugin (e.g. http://\<your-mozart-instance\>:9200/_plugin/head)*

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-head.png|alt=head]]
    Alternatively, you can query for the the "Hello World" job-spec using curl
    ```
    curl -s localhost:9200/job_specs/job_spec/job-hello_world:master | python -m json.tool
    {
      "_id": "job-hello_world:master",
      "_index": "job_specs",
      "_source": {
        "command": "/home/ops/verdi/ops/hello_world/run_hello_world.sh",
        "container": "container-hello_world:master",
        "id": "job-hello_world:master",
        "job-version": "master",
        "params": [],
        "recommended-queues": [
          "job_worker-small"
        ],
        "resource": "jobspec"
      },
      "_type": "job_spec",
      "_version": 1,
      "found": true
    }
    ```
17. Now let's try to run our hello_world job type using the Dataset Faceted Search interface (aka `tosca`) running on the `grq` instance. First, let's ingest a dataset so that we have at least 1 dataset in our catalog. On the `mozart` instance run
    ```
    source ~/mozart/bin/activate
    cd ~/.sds/files
    ~/mozart/ops/hysds/scripts/ingest_dataset.py AOI_sacramento_valley ~/mozart/etc/datasets.json
    ```
18. Navigate to your `grq` instance (e.g. http://\<your-grq-instance\>/search)* to validate that the AOI dataset was ingested

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-ingest.png|alt=ingest]]

19. Next click on the "On-Demand" button. If you don't see the "On-Demand" button, then the AOI dataset wasn't ingested correctly. Otherwise, a modal titled "On-Demand (Process Results)" should pop up as below

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-on_demand.png|alt=on-demand]]

    Enter the value "test" into the "Tag" field, select "Hello World [master]" for the "Action" field, and select "job_worker-small" for the "Queue" field (should already be selected because it is the "recommended-queue" based on our job-spec.json.hello_world that we composed earlier). You can leave the "Priority" field at "0" for now

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-submit_on_demand.png|alt=submit]]

    Click on "Process Now".
20. You will then get another modal providing you with a link to the HySDS Resource Management interface (aka `figaro`) for the job you just submitted. Click on the link to open up `figaro`. If the link doesn't open up correctly, there may be a routing issue with the private vs. public IP address in the `tosca` configuration. Alternately, navigate to your `mozart` instance (e.g. http://\<your-mozart-instance\>/figaro)* to view your job. Click on the "Queued" button to facet down to the jobs that are in a "job-queued" state

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-job_queued.png|alt=queued]]

    For advanced users, navigate to the RabbitMQ Admin interface (e.g. http://\<your-mozart-instance\>:15672/#/queues) 
 running on your `mozart` instance to view the queued jobs and tasks

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-rabbitmq.png|alt=rabbitmq]]

    It will be stuck in the `job-queued` because there are no workers configured to process jobs from the "job_worker-small" queue. 
21. Let's configure your `factotum` instance to start up a worker that will pull jobs from the "job_worker-small" queue. What we will do is copy the default supervisord.conf template for factotum to `~/.sds/files`, and modify it to include start up of `celery` workers for the `job_worker-small` queue:
    ```
    cd ~/.sds/files
    cp ~/mozart/ops/hysds/configs/supervisor/supervisord.conf.factotum .
    ```
    Edit the `supervisord.conf.factotum` file
    ```
    vi ~/.sds/files/supervisord.conf.factotum
    ```
    append the following lines to the bottom of the file and save
    ```
    [program:job_worker-small]
    directory={{ OPS_HOME }}/verdi/ops/hysds
    environment=HYSDS_ROOT_WORK_DIR="/data/work",
                HYSDS_DATASETS_CFG="{{ OPS_HOME }}/verdi/etc/datasets.json"
    command=celery worker --app=hysds --concurrency=1 --loglevel=INFO -Q job_worker-small -n %(program_name)s.%(process_num)02d.%%h -O fair --without-mingle --without-gossip --heartbeat-interval=60
    process_name=%(program_name)s-%(process_num)02d
    priority=1
    numprocs=1
    numprocs_start=0
    redirect_stderr=true
    stdout_logfile=%(here)s/../log/%(program_name)s-%(process_num)02d.log
    startsecs=10
    ```
22. Since there are no jobs running in your cluster, we can safely update the factotum instance
    ```
    sds stop factotum
    ```
    Check the status of `supervisord` on factotum
    ```
    sds status factotum
    ```
    We're ready to update if it shows
    ```
    ########################################
    factotum
    ########################################
    [100.64.106.64] Executing task 'status'
    Supervisord is not running on factotum.
    ```
    Finally update and start up factotum:
    ```
    sds update factotum
    sds start factotum
    ```
    Rerun `sds status factotum` to verify that your new worker for `job_worker-small` is `RUNNING`
    ```
    [172.31.6.173] out: job_worker-small:job_worker-small-00       RUNNING   pid 20200, uptime 0:00:25
    ```
23. Navigate to your `mozart` instance (e.g. http://\<your-mozart-instance\>/figaro)* to view your job again. Click on the "Started" button to facet down to the jobs that are in a "job-started" state

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-job_started.png|alt=started]]

    When the job is finished executing, you should have a job that is in a "job-completed" state

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-job_completed.png|alt=completed]]

24. Click on the "work directory" link to view the live work directory on factotum. All STDOUT output will go into the `_stdout.txt` file and all STDERR output will go into the `_stderr.txt`. Here's what `_stdout.txt` should show

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-stdout.png|alt=stdout]]

    and what the `_stderr.txt` should show

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-stderr.png|alt=stderr]]

    ### Note: 
     If you run into issues like:
    ```
    Failed to download http://<aws product bucket url>/products/incoming/v0.1/<file>: 404 Client Error: Not Found for url: …
    ```
    it could be due to permission issue in aws. Check the permission tab in 'grfn-v2-ops-product-bucket' to ensure you have the same configuration, specially, the policy file (dont forget to edit the bucket name in your policy file). Also download the index.html file from there and add to your bucket (do necessary edits to reflect your bucket)
### That's it!
Congratulations, you've integrated a new job type into your HySDS cluster and successfully executed it.

### Next Steps
Now that you've integrated your first job type, continue on to **[[Hello Dataset]]** to integrate a dataset produced by this job type.


_____________________________________________________________________________________
*With the OPS_USER and OPS_PASSWORD, you can use the https (e.g. https:// ) protocol to gain access to the url.