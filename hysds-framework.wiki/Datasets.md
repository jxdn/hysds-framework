# Dataset Specification (how to author a new dataset type)
In order to for HySDS to recognize a dataset, the dataset must follow certain conventions.  These conventions are documented on this page and must be implemented by the PGE or the PGE wrapper.

## Dataset ID
Each product should have a dataset ID.  This name is used to determine the type of the dataset and name all the important files for the dataset.  A dataset ID is matched against entries found in the `datasets.json` file to determine its type.  
In this example, we shall use the dataset ID `dumby-product-20170101T000000Z-3lx0a`.

## Directory
Any directory containing the below JSON files and found within the working directory supplied to the PGE is considered a dataset. Thus this directory must be named with the dataset's ID (see above):
```
$ pwd
/data/work/example_work_dir/dumby-product-20170101T000000Z-3lx0a
$ ls
dumby-product-20170101T000000Z-3lx0a.dataset.json
dumby-product-20170101T000000Z-3lx0a.met.json
dumby-product-20170101T000000Z-3lx0a.prov_es.json
dumby-product-20170101T000000Z-3lx0a.h5
pge_output_2.h5
errors.txt
other_metadata.xml
```
_Note that any other PGE data files should be placed in the \<Dataset ID\> directory, as the whole directory is the dataset._

## HySDS dataset and metadata JSON files
### dataset JSON file
A product must produce a \<Dataset ID\>.dataset.json in the \<Dataset ID\> directory.  This file contains JSON formatted metadata representing the cataloged dataset metadata:
```
$ cat dumby-product-20170101T000000Z-3lx0a.dataset.json
 {
  "version": "v1.0",
  "label": "dumby product for 2017-01-01T00:00:00Z",
  "location": {
    "type": "polygon",
    "coordinates": [
      [
        [-122.9059682940358,40.47090915967475],
        [-121.6679748715316,37.84406528996276],
        [-120.7310161872557,38.28728069813177],
        [-121.7043611684245,39.94137004454238],
        [-121.9536916840953,40.67097860759095],
        [-122.3100379696548,40.7267890636145],
        [-122.7640648263371,40.5457010812299],
        [-122.9059682940358,40.47090915967475]
      ]
    ]
  },
  "starttime": "2017-01-01T00:00:00",
  "endtime": "2017-01-01T00:05:00"
}
```
The required fields are:
- `version`

The optional fields are:
- `label`
- `location` (in GeoJSON format)
- `starttime`
- `endtime`

### metadata JSON file
In addition, other metadata data can be added to a \<Dataset ID\>.met.json in the \<Dataset ID\> directory. As long as the file conforms to the JSON format, the dataset developer has free reign on what goes into this file:
```
$ cat dumby-product-20170101T000000Z-3lx0a.met.json
{
  "startingRange": 800026.4431219272,
  "sensor": "SAR-C Sentinel1",
  "esd_threshold": 0.85,
  "tiles": true,
  "reference": true,
  "trackNumber": 144,
  "lookDirection": "right",
  "beamMode": "IW",
  "direction": "descending",
  "inputFile": "sentinel.ini",
  "polarization": "VV",
  "imageCorners": {
    "maxLon": -117.56055555555555,
    "minLon": -119.06166666666667,
    "minLat": 35.92333333333333,
    "maxLat": 37.766666666666666
  },

...

  "orbitRepeat": 175
}
```

### PROV-ES JSON file (optional)
If the developer has instrumented the PGE to use the PROV-ES (Provenence for Earth Science) service, a \<Dataset ID\>.prov_es.json file must exist in the \<Dataset ID\> directory containing the PROV-ES JSON:
```
$ cat dumby-product-20170101T000000Z-3lx0a.prov_es.json
{
  "wasAssociatedWith": {
    "hysds:a379e0e0-32ac-59b7-89b8-80045bad1ba0": {
      "prov:agent": "hysds:47bce551-6fac-5bb5-adcc-61918c608f96", 
      "prov:activity": "hysds:8e5a0556-99b5-56a9-ae0b-e1de3f5fe216"
    }
  }, 
  "used": {
    "hysds:d04421e2-9847-5073-9f45-1b3e0b011cd8": {
      "prov:role": "input", 
      "prov:time": "2015-09-13T14:18:39.037508+00:00", 
      "prov:entity": "hysds:e08c901b-2f98-514a-98cc-76f5a124ccd6", 
      "prov:activity": "hysds:8e5a0556-99b5-56a9-ae0b-e1de3f5fe216"
    }
  }, 
  "agent": {
    "eos:JPL": {
      "prov:type": {
        "type": "prov:QualifiedName", 
        "$": "prov:Organization"
      }, 
      "prov:label": "Jet Propulsion Laboratory"
    }, 
    "hysds:47bce551-6fac-5bb5-adcc-61918c608f96": {
      "hysds:host": "x.x.x.x", 
      "prov:label": "hysds:pge_wrapper/x.x.x.x/20202/2015-09-13T14:18:39.866510", 
      "prov:type": {
        "type": "prov:QualifiedName", 
        "$": "prov:SoftwareAgent"
      }, 
      "hysds:pid": "20202"
    },

...

  "prefix": {
    "info": "http://info-uri.info/", 
    "bibo": "http://purl.org/ontology/bibo/", 
    "hysds": "http://hysds.jpl.nasa.gov/hysds/0.1#", 
    "eos": "http://nasa.gov/eos.owl#", 
    "gcis": "http://data.globalchange.gov/gcis.owl#", 
    "dcterms": "http://purl.org/dc/terms/"
  }, 
  "activity": {
    "hysds:8e5a0556-99b5-56a9-ae0b-e1de3f5fe216": {
      "eos:runtimeContext": "hysds:runtimeContext-ariamh-trigger_pietro_use_case_triggersf", 
      "hysds:job_url": "http://x.x.x.x:8085/jobs/2015/09/13/create_interferogram-CSKS2_RAW_HI_04_HH_RA_20110811135636_20110811135643.interferogram.json_10-20150913T072449.918362Z", 
      "prov:startTime": "2015-09-13T13:36:29.047370+00:00", 
      "prov:type": "hysds:create_interferogram", 
      "hysds:mozart_url": "https://x.x.x.x:8888", 
      "eos:usesSoftware": [
        "eos:ISCE-2.0.0_201410"
      ], 
      "hysds:job_type": "create_interferogram", 
      "hysds:job_id": "create_interferogram-CSKS2_RAW_HI_04_HH_RA_20110811135636_20110811135643.interferogram.json_10-20150913T072449.918362Z", 
      "prov:endTime": "2015-09-13T14:18:39.835057+00:00", 
      "prov:label": "create_interferogram-2015-09-13T14:18:39.037508Z"
    }
  }, 
  "wasGeneratedBy": {
    "hysds:bc3f219a-4002-57d8-8151-9fe3b7776d40": {
      "prov:role": "output", 
      "prov:time": "2015-09-13T14:18:39.037508+00:00", 
      "prov:entity": "hysds:6534285c-01f0-5070-8294-a3847b048d5b", 
      "prov:activity": "hysds:8e5a0556-99b5-56a9-ae0b-e1de3f5fe216"
    }
  }
}
```

# Dataset Configuration (how to configure HySDS to recognize a dataset for ingest)
Once a PGE is configured to generate a dataset according to the HySDS dataset specification above, users will have to configure HySDS to recognize the new dataset type so that they get ingested properly into the system.

## datasets JSON config
The `datasets.json` configuration file is centrally stored at some location (usually on the mozart instance in the ops user's `~/.sds/files` directory) and is published to a repository where instantiating workers can pull from. At job execution time, the `datasets.json` configuration file is placed in the work directory of the job for the dataset recognition step. The format for `datasets.json` is:
```
{
  "datasets": [
    {
      "ipath": "hysds::data/dumby-product",
      "version": "v0.1",
      "level": "l0",
      "type": "dumby-data",
      "id_template": "${id}",
      "match_pattern": "/(?P<id>dumby-product-\\d+)$",
      "alt_match_pattern": null,
      "extractor": null,
      "publish": {
        "s3-profile-name": "<aws creds profile name>",
        "location": "<S3 url to S3 bucket>/products/dumby/${id}",
        "urls": [
          "<HTTP url to S3 bucket>/products/dumby/${id}",
          "<S3 url to S3 bucket>/products/dumby/${id}"
        ]
      },
      "browse": {
        "s3-profile-name": "<aws creds profile name>",
        "location": "<S3 url to S3 bucket>/browse/dumby/${id}",
        "urls": [
          "<HTTP URL to S3 bucket>/browse/dumby/${id}",
          "<S3 URL to S3 bucket>/browse/dumby/${id}"
        ]
      }
    }
  ]
}
```

At its core the file is a list of data set objects associated with the key "datasets".
Each data set object specifies the following properties:
- `ipath`: a namespaced name for the product type
- `version`: a version string for the product
- `level`: a level (L0, L1, L2, L3 ...) string for the product
- `type`: type name for the data product
- `id_template`: a template to build the id for the product from capture groups from in the match_pattern (#6)
- `match_pattern`: regular expression to match product name strings (from sub-directories in PGE working directory).  Should always start with a '/'.
- `alt_match_pattern`: alternate pattern to #6 (usually just null
- `extractor`: path to executable binary or script to run further metadata extraction on the dataset
- `publish`: an object specifying how to publish this product composed of the following fields
- `s3-profile-name`: AWS profile to use as defined in `~/.aws`.
- `location`: Location for data-store location of final archived product using Osaka prefix
- `urls`: List of urls pointing to the product once archived.  Can include http access, s3 access or any other URL type
- `browse`:  an object with the same attributes as publish (#9) but handling the publishing of browse-images

For example, the `datasets.json` file below shows the configuration for our dumby dataset (`dumby-product-20170101T000000Z-3lx0a`):
```
{
  "datasets": [
    {
      "ipath": "hysds::data/dumby-product",
      "match_pattern": "/(?P<id>dumby-product-\\d+)$",
      "alt_match_pattern": null,
      "extractor": null,
      "level": "l0",
      "type": "dumby-data",
      "publish": {
        "s3-profile-name": "default",
        "location": "s3://s3-us-west-2.amazonaws.com:80/my_bucket/products/dumby/${version}/${id}",
        "urls": [
          "http://my_bucket.s3-website-us-west-2.amazonaws.com/products/dumby/${version}/${id}",
          "s3://s3-us-west-2.amazonaws.com:80/my_bucket/products/dumby/${version}/${id}"
        ]
      },
      "browse": {
        "s3-profile-name": "default",
        "location": "s3://s3-us-west-2.amazonaws.com:80/my_bucket/browse/dumby/${version}/${id}",
        "urls": [
          "http://my_bucket.s3-website-us-west-2.amazonaws.com/browse/dumby/${version}/${id}",
          "s3://s3-us-west-2.amazonaws.com:80/my_bucket/browse/dumby/${version}/${id}"
        ]
      }
    }
  ]
}
