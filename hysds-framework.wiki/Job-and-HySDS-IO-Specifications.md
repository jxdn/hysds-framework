HySDS provides capabilities to automatically manage, and continuously integrate the jobs that are in the system. In order to do that, it separates job and interface. The `job-spec` is used to specify a job type and the `hysds-io` is used to specify a job type's interface of input/output parameters. HySDS utilizes docker to encapsulate job types and execute them on workers.

## Job Specification (job-spec.json.*)
The `job-spec` contains the job specific information and points to the container specification that can run this job. There can be many job specifications to a single container specification.

The job specification is a JSON field and requires several metadata fields in order for the job to be used in the HySDS system.

### JSON Parameters
| key | constraint | type | description |
| --- | ---------- | ---- | ----------- |
| `command` | required | string | executable path inside container |
| `params` | required | array | list of param objects required to run this job (see "`param` Object" below) |
| `imported_worker_files` | optional | object | mapping of host file/directory into container (see "`imported_worker_files` Object" below) |
| `dependency_images` | optional | array | list of dependency image objects (see "`dependency_image` Object" below) |
| `recommended-queues` | optional | array | list of recommended queues |
| `disk_usage` | optional | string | minimum free disk usage required to run job specified as "\\d+(GB\|MB\|KB)", e.g. "100GB", "20MB", "10KB" |
| `soft_time_limit` | optional | int | soft execution time limit in seconds; worker will send a catchable exception to task to allow for cleanup before being killed |
| `time_limit` | optional | int | hard execution time limit in seconds; worker will send an uncatchable exception to the task and will force terminate it |
| `pre` | optional | array | list of strings specifying pre-processor functions to run; behavior depends on `disable_pre_builtins`; more info on [Pre-processor Functions](Preprocessor-and-Postprocessor-Functions) |
| `disable_pre_builtins` | optional | boolean | if set to true, default builtin pre-processors (currently [`hysds.utils.localize_urls`]) are disabled and would need to be specified in `pre` to run; if not specified or set to false, list of pre-processors specified by `pre` is appended after the default builtins |
| `post` | optional | array | list of strings specifying post-processor functions to run; behavior depends on `disable_post_builtins`; more info on [Post-processor Functions](Preprocessor-and-Postprocessor-Functions) |
| `disable_post_builtins` | optional | boolean | if set to true, default builtin post-processors (currently [`hysds.utils.publish_datasets`]) are disabled and would need to be specified in `post` to run; if not specified or set to false, list of post-processors specified by `post` is appended after the default builtins |

#### `param` Object
| key | constraint | type | description |
| --- | ---------- | ---- | ----------- |
| `name` | required | string | parameter name |
| `destination` | required | string | `positional` - for command line arguments, `localize` - for automatically downloading files/directories, `context` - to add information to _context.json |

#### `imported_worker_files` Object
| key | key type | value | value type |
| --- | -------- | ----- | ---------- |
| path to file or directory on host | string | path to file or directory in container | string |
| path to file or directory on host | string | one item list of path to file or directory in container | array |
| path to file or directory on host | string | two item list of path to file or directory in container and mount mode: `ro` for read-only and `rw` for read-write (default is `ro`) | array |

#### `dependency_image` Object
| key | constraint | type | description |
| --- | ---------- | ---- | ----------- |
| `container_image_name` | required | string | docker image name and tag, e.g. docker.io/centos:7 |
| `container_image_url` | optional | string | url to docker image location or null; if null or unspecified, image will be pulled from dockerhub e.g. "docker pull docker.io/centos:7" |
| `container_mappings` | optional | object | same as `imported_worker_files` object |

### Syntax
```
{
  "command": "string",
  "recommended-queues": [ "string" ],
  "disk_usage":"\d+(GB|MB|KB)",
  "imported_worker_files": {
    "string": "string",
    "string": [ "string" ],
    "string": [ "string", "ro" | "rw" ]
  },
  "dependency_images": [
    {
      "container_image_name": "string",
      "container_image_url": "string" | null,
      "container_mappings": {
        "string": "string",
        "string": [ "string" ],
        "string": [ "string", "ro" | "rw" ]
      }
    }
  ],
  "soft_time_limit": int,
  "time_limit": int,
  "disable_pre_builtins": true | false,
  "pre": [ "string" ],
  "disable_post_builtins": true | false,
  "post": [ "string" ],
  "params": [
    {
      "name": "string",
      "destination": "positional" | "localize" | "context"
    }
  ]
}
```

### Example
```
{
  "command": "/home/ops/verdi/ops/hello_world/run_hello_world.sh",
  "recommended-queues": [ "factotum-job_worker-small" ],
  "disk_usage":"10MB",
  "imported_worker_files": {
    "$HOME/.netrc": ["/home/ops/.netrc"],
    "$HOME/.aws": ["/home/ops/.aws", "ro"],
    "$HOME/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "dependency_images": [
    {
      "container_image_name": "aria/isce_giant:latest",
      "container_image_url": "s3://s3-us-west-2.amazonaws.com/my-hysds-code-bucket/aria-isce_giant-latest.tar.gz",
      "container_mappings": {
        "$HOME/.netrc": ["/root/.netrc"],
        "$HOME/.aws": ["/root/.aws", "ro"]
      }
    }
  ],
  "soft_time_limit": 900,
  "time_limit": 960,
  "disable_pre_builtins": false,
  "pre": [ "some.python.preprocessor.function" ],
  "disable_post_builtins": false,
  "post": [ "hysds.utils.triage" ],
  "params": [
    {
      "name": "localize_url",
      "destination": "localize"
    },
    {
      "name": "file",
      "destination": "positional"
    },
    {
      "name": "prod_name",
      "destination": "context"
    }
  ]
}
```

## HySDS IO Specification (hysds-io.json.*)
The `hysds-io` file is designed to create a wiring to the specified `params` defined in the `job-spec`. It specifies the from-where as opposed to the to-where, which is specified by the job-spec. It has several fields defined in its JSON syntax.

### JSON Parameters
| key | constraint | type | description |
| --- | ---------- | ---- | ----------- |
| `label` | optional | string | label to be used when this job type is displayed in web interfaces (`tosca` and `figaro`); otherwise it will show an automatically generated label based on the string after the "hysds.io.json." of the `hysds-io` file |
| `submission_type` | optional | string | specifies if the job should be submitted once per product in query or once per job submission; `iteration` for a submission of the job for each result in the query or `individual` for a single submission; defaults to `iteration` |
| `action-type` | optional | string | action type to expose job as; `on-demand`, `trigger`, or `both`; defaults to `both` |
| `allowed_accounts` | optional | array | list of strings specifying account user IDs allowed to run this job type from the web interfaces (`tosca` and `figaro`); if not defined, ops account is the only account that can access this job type; if `_all` is specified in the list, then all users will be able to access this job type |
| `params` | required | array | list of matching param objects from `job-spec` required to run this job (see "`param` Object" below); |

#### A Note on `submission_type`: `iteration` and `individual`
Iteration jobs submit one job for each of the faceted results in the selected set (`tosca` or `figaro`). This means that each parameter using "dataset_jpath:" (see below) submits a unique job where the value of that parameter corresponds to an individual product from the faceted set.

Individual jobs simply submit one job, no matter how many products are faceted.  If "dataset_jpath:" is specified, the corresponding values are aggregated into a list, and that becomes the value for the parameter below.

#### `params` Object
| key | constraint | type | description |
| --- | ---------- | ---- | ----------- |
| `name` | required | string | parameter name; should match corresponding parameter name in `job-spec` |
| `from` | required | string | `value` for hard-coded parameter value, `submitter` for user submitted value, `dataset_jpath:<jpath>` to extract value from ElasticSearch result using a JPath-like notation (see "`from` Specification" below) |
| `value` | required if `from` is set to `value` | string | hard-coded parameter value |
| `type` | optional | string | possible values: `text`, `number`, `datetime`, `date`, `boolean`, `enum`, `email`, `textarea`, `region`, `container_version`, `jobspec_version`, `hysdsio_version` (see "Valid Param Types" below) |
| `default` | optional | string | default value to use (must be string even if it's a number) |
| `optional` | optional | boolean | parameter is optional and can be left blank |
| `placeholder` | optional | string | value to use as a hint when displaying the form input |
| `enumerables` | required if `type` is `enum` | array | list of string values to enumerate via a dropdown list in the form input |
| `lambda` | optional | string | a lambda function to process the value during submission |
| `version_regex` | required if `type` is `container_version`, `jobspec_version` or `hysdsio_version` | string | regex to use to filter on front component of respective container, `job-spec` or `hysds-io` ID; e.g. if `type`=`jobspec_version`, `version_regex`=`job-time-series` and list of all `job-specs` IDs in the system is ["job-time-series:release-20171103", "job-hello_world:release-20171103", job-time-series:master"], the list will be filtered down to those whose ID matched the regex `job-time-series` in the front component (string before the first ":"); in this example the resulting filtered set of release tags/branches will be listed as ["release-20171103", "master"] in a dropdown box; similar to `type`=`enum` and `enumerable` set to all release tags for a certain job type |

#### `from` Specification
| value | descripton |
| ----- | ---------- |
| `value` | hard-coded parameter value found in `value` field in hysds-io.json |
| `submitter` | comes from operator or submitter of job; usually displayed as input boxes on the web UI forms for `tosca` and `figaro` |
| `dataset_jpath:<jpath>` | comes from an ElasticSearch result, e.g. in `tosca`, the iterated dataset's metadata supplies this field; metadata is mined from the metadata found at the "." separated path to a given metadata field. i.e. metadata.field1 pulls from the metadata field's field1 value; the top level of the jpath is the top of the ES entry. i.e. dataset_jpath:_id is a valid dataset jpath to pull out ES's ID |

#### Valid Param Types
| value | descripton |
| ----- | ---------- |
| `text` | a text string, will be kept as text |
| `number` | a real number |
| `date` | a date in ISO8601 format: `YYYY-MM-DD`; will be treated as a "text" field for passing into the container |
| `datetime` | a date with time in ISO8601 format: `YYYY-MM-DDTHH:mm:SS.SSS`; will be treated as a "text" field |
| `boolean` | true or false in a drop down |
| `enum` | one of a set of options in a drop down; must specify `enumerables` field to specify the list of possible options; these will be "text" types in the enumerables set |
| `email` | an e-mail address, will be treated as "text" |
| `textarea` | same as text, but displayed larger with the textarea HTML tag |
| `region` | auto-populated from the facet view leaflet tool |
| `container_version` | a version of an existing container registered in the Mozart API; must define "version_regex" field |
| `jobspec_version` | a version of an existing `job-spec` registered in the Mozart API; must define "version_regex" field |
| `hysdsio_version` | a version of an existing `hysds-io` registered in the Mozart API; must define "version_regex" field |

#### A note on type conversion and default values
Due to a limitation in ElasticSearch mappings, hysds-io "default_values" should be represented as JSON strings.  This parallels the input from the user which will come through as a string. Internally the system will convert the value into its "type" within python.  This happens just before the call to a "lambda" function and thus the user can expect proper types inside the lambda function.

Certain types default to python strings. these types include: "container_version", "hysdsio_version", "jobspec_version", "date", "datetime", "text", "string", "textbox", "enum", and "email".

All numbers are converted into floats due to the nature of JSON which does not distinguish between integer and float values.  If a python integer is needed by the job, supply a lambda like so "lambda x: int(x)".

### Syntax
```
{
  "label": "string",
  "submission_type": "individual" | "iteration",
  "allowed_accounts": [ "string" ],
  "params": [
    {
      "name": "string",
      "from": "value" | "submitter" | "dataset_jpath:<path>",
      "type": "string",
      "default": "string",
      "placeholder": "string",
      "enumerables": [ "string" ],
      "lambda": "string",
      "value": "string",
      "version_regex": "string"
    }
  ]
}
```

### Example
```
{
  "label": "Hello World",
  "submission_type": "iteration",
  "allowed_accounts": [ "ops" ],
  "params": [
    {
      "name": "localize_url",
      "from": "dataset_jpath:_source",
      "lambda": "lambda ds: \"%s/%s\" % ((filter(lambda x: x.startswith('s3://'), ds['urls']) or ds['urls'])[0], ds['metadata']['data_product_name'])"
    },
    {
      "name": "file",
      "from": "dataset_jpath:_source.metadata.data_product_name"
    },
    {
      "name": "prod_name",
      "from": "dataset_jpath:_source.metadata.prod_name"
    }
  ]
}
```
