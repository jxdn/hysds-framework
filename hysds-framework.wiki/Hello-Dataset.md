# Integrating a "Hello World" Dataset Type
For definitions of terminology used, please refer to our [terminology reference](https://github.com/hysds/hysds-framework/wiki/Docs#general-hysds-terminology).

In this tutorial, we continue where we left off in integrating our first job type, [[Hello World]]. We will update our job type to generate a HySDS dataset that will be automatically ingested and catalogued into the system.

1. On the `mozart` instance, navigate to the `hello_world` repository
   ```
   cd ~/mozart/ops/hello_world
   ```
2. We need to author a dataset ID scheme according to the [HySDS dataset specification](Datasets#dataset-specification-how-to-author-a-new-dataset-type). In this tutorial, we will use the following dataset ID scheme
   ```
   hello_world-product-<year><month><day>T<hour><minute><second>Z-<hash>
   ```
   where `<year>` will be replaced by a fictional observation year and likewise for `<month>`, `<day>`, `<hour>`, `<minute>`, `<second>` and `<hash>`. An example instance of this dataset ID would be
   ```
   hello_world-product-20170901T000102.021Z-a3d0x
   ```
3. Edit the script that runs our job type
   ```
   vi run_hello_world.sh
   ```
   and add code to create an instance of our dataset. The script should look like this
   ```
   #!/bin/bash
   echo "Hello World"
   echo "Hello World to STDERR" 1>&2

   # generate dataset ID
   timestamp=$(date -u +%Y%m%dT%H%M%S.%NZ)
   hash=$(echo $timestamp | sha224sum | cut -c1-5)
   id=hello_world-product-${timestamp}-${hash}
   echo "dataset ID: $id"

   # create dataset directory
   mkdir $id

   # create fake data
   fake_data_file=${id}/fake_data.dat
   dd if=/dev/urandom of=$fake_data_file bs=1M count=5

   # create minimal dataset JSON file
   dataset_json_file=${id}/${id}.dataset.json
   echo "{\"version\": \"v1.0\"}" > $dataset_json_file

   # create minimal metadata file
   metadata_json_file=${id}/${id}.met.json
   echo "{}" > $metadata_json_file
   ```
   Access the documentation on the specifications for the HySDS [dataset JSON file](Datasets#dataset-json-file) and [metadata JSON file](Datasets#metadata-json-file) for more detail.
   Finally, commit your change and push the update
   ```
   git commit -a -m "add dataset generation"
   git push
   ```
   If your jenkins job for the master branch of your `hello_world` repo was configured correctly, the `git push` should have kicked off a rebuild of the container. If not, navigate to your `ci` instance (e.g. http://<your-ci-instance>:8080) and manually schedule a build

   [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-jenkins.png|alt=jenkins]]

4. Test the script by running it
   ```
   ./run_hello_world.sh  
   dataset ID: hello_world-product-20170907T141956.052530118Z-8d5a7
   5+0 records in
   5+0 records out
   5242880 bytes (5.2 MB) copied, 0.439011 s, 11.9 MB/s
   ```
   You should see the dataset directory you just created sitting there in your directory. Ensure that the `fake_data.dat` and the HySDS dataset and metadata JSON files were generated
   ```
   ls -l hello_world-product-*
   total 5128
   -rw-r--r-- 1 ops ops 5242880 Sep  7 14:19 fake_data.dat
   -rw-r--r-- 1 ops ops      20 Sep  7 14:19 hello_world-product-20170907T141956.052530118Z-8d5a7.dataset.json
   -rw-r--r-- 1 ops ops       3 Sep  7 14:19 hello_world-product-20170907T141956.052530118Z-8d5a7.met.json
   ```
5. Now let's try to ingest this new dataset into our catalog
   ```
   ~/mozart/ops/hysds/scripts/ingest_dataset.py hello_world-product-20170907T141956.052530118Z-8d5a7 ~/mozart/etc/datasets.json
   ```
   You should've gotten an error that the data was not recognized
   ```
   Traceback (most recent call last):
     File "/home/ops/mozart/ops/hysds/scripts/ingest_dataset.py", line 16, in <module>
       os.path.abspath(args.ds_dir), None)
     File "/home/ops/mozart/ops/hysds/hysds/dataset_ingest.py", line 204, in ingest
       r = Recognizer(dsets_file, local_prod_path, objectid, version)
     File "/home/ops/mozart/ops/hysds/hysds/recognize.py", line 53, in __init__
       self._recognize(path)
     File "/home/ops/mozart/ops/hysds/hysds/recognize.py", line 84, in _recognize
       raise RecognizerError("No dataset configured for %s. Check %s." % (path, self.dataset_file))
   hysds.recognize.RecognizerError: No dataset configured for /home/ops/mozart/ops/hello_world/hello_world-product-20170907T141956.052530118Z-8d5a7. Check /home/ops/mozart/etc/datasets.json.
   ```
6. Let's configure our new dataset so that it can be recognized by our HySDS cluster. Since we want to ensure all nodes in our cluster is able to recognize this new dataset, we need to make updates to the `datasets.json` file in `~/.sds/files`. Still on `mozart`
   ```
   cd ~/.sds/files
   ```
   edit the datasets.json file
   ```
   vi datasets.json
   ```
   and add the following dataset configuration below the `dumby-product` configuration
   ```
    { 
      "ipath": "hysds::data/hello_world-product",
      "match_pattern": "/(?P<id>hello_world-product-(?P<year>\\d{4})(?P<month>\\d{2})(?P<day>\\d{2})T.*)$",
      "alt_match_pattern": null,
      "extractor": null,
      "level": "l0",
      "type": "hello_world-data",
      "publish": {
        "s3-profile-name": "default",
        "location": "s3://{{ DATASET_S3_ENDPOINT }}:80/{{ DATASET_BUCKET }}/products/hello_world/{version}/{year}/{month}/{day}/{id}",
        "urls": [
          "http://{{ DATASET_BUCKET }}.{{ DATASET_S3_WEBSITE_ENDPOINT }}/products/hello_world/{version}/{year}/{month}/{day}/{id}",
          "s3://{{ DATASET_S3_ENDPOINT }}:80/{{ DATASET_BUCKET }}/products/hello_world/{version}/{year}/{month}/{day}/{id}"
        ]
      },
      "browse": {
        "s3-profile-name": "default",
        "location": "s3://{{ DATASET_S3_ENDPOINT }}:80/{{ DATASET_BUCKET }}/browse/hello_world/{version}/{year}/{month}/{day}/{id}",
        "urls": [
          "http://{{ DATASET_BUCKET }}.{{ DATASET_S3_WEBSITE_ENDPOINT }}/browse/hello_world/{version}/{year}/{month}/{day}/{id}",
          "s3://{{ DATASET_S3_ENDPOINT }}:80/{{ DATASET_BUCKET }}/browse/hello_world/{version}/{year}/{month}/{day}/{id}"
        ]
      }
    }
   ```
   and save.
7. Run the following fabric command to push the update you made to the `datasets.json` template on `mozart`
   ```
   cd ..
   fab -f cluster.py -R mozart send_template:datasets.json,~/mozart/etc/datasets.json
   ```
   You should have output similar to the following
   ```
   [172.31.4.210] Executing task 'send_template'
   [172.31.4.210] run: cp "$(echo ~/mozart/etc/datasets.json)"{,.bak}
   [172.31.4.210] put: <file obj> -> /home/ops/mozart/etc/datasets.json

   Done.
   ```
8. Now let's try to ingest this new dataset into our catalog
   ```
   cd ~/mozart/ops/hello_world
   ~/mozart/ops/hysds/scripts/ingest_dataset.py hello_world-product-20170907T141956.052530118Z-8d5a7 ~/mozart/etc/datasets.json
   ```
9. Verify that your new dataset was published to your HySDS cluster. Open up a browser and point it to your `grq` instance's `tosca` (Dataset FacetSearch) interface (e.g. https://<your-grq-instance>/search). On the left panel you should see your new dataset type listed under the "dataset" facet showing 1 result

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-tosca.png|alt=tosca]]

    Click on "hello_world-product" under the "dataset" facet and your result set should be constrained now to just the dataset you just ingested

    [[https://github.com/hysds/hysds-framework/blob/master/wiki_images/hello_world-dataset_ingest.png|alt=dataset]]
10. Now that we've verified that the new dataset is recognized by the system, let's push the updates to the rest of the system. On `mozart`
    ```
    sds stop all
    sds update all
    sds ship
    sds start all
    ```

### That's it!
Congratulations, you've integrated a new dataset type into your HySDS cluster.

To configure Autoscaling groups for your HySDS cluster, continue on to **[[Create-AWS-Autoscaling-Group-for-Verdi]]**.

To configure a staging area for your HySDS cluster, continue on to **[[Create-AWS-Resources-for-Staging-Area]]**.
