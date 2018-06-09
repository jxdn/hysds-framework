HySDS provides a plugin facility to hook in arbitrary function calls before (`pre-processor` functions) and after (`post-processor` functions) the execution of a docker command in a job. HySDS provides a builtin pre-processor function that runs by default for each job (`hysds.utils.localize_urls`) to localize any inputs specified in the job payload. It also provides a builtin post-processor function that runs by default as well (`hysds.utils.publish_datasets`) to search the work directory for any HySDS datasets to publish.

## Function Definition
Pre-processor and post-processor functions are required to take 2 arguments: the job dict and context dict. These functions can do what they need using information stored in the job and context dicts. The function must return a boolean result, either `True` or `False`. In the case of pre-processor functions, the docker command for the job is only executed if all pre-processor functions return a `True`. In the case of post-processor functions, any that return a `False` will be logged but the job will not be considered a failure. In both cases, if the function raises an exception, then the job is a failure.

## Pre-processor Functions
By default, `job-specs` have an implicit pre-processor function defined, `hysds.utils.localize_urls`. Thus the following `job-spec`:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

implicitly provides the "pre" parameter as follows:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "pre": [ "hysds.utils.localize_urls" ],
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

To disable this builtin feature, specify the "disable_pre_builtins" parameter as follows:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "disable_pre_builtins": true,
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

In this case, even though there exists a parameter "uri" with destination "localize", the `hysds.utils.localize_urls` pre-processor will not run. To specify additional pre-processor functions, define them using the "pre" parameter like so:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "pre": [ "my.custom.preprocessor_function" ],
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

Because the default builtin pre-processor functions aren't disabled, the effect of the above `job-spec` is this:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "disable_pre_builtins": true,
  "pre": [ "hysds.utils.localize_urls", "my.custom.preprocessor_function" ],
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

## Post-processor Functions
By default, `job-specs` have an implicit post-processor function defined, `hysds.utils.publish_datasets`. Thus the following `job-spec`:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

implicitly provides the "post" parameter as follows:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "post": [ "hysds.utils.publish_datasets" ],
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

To disable this builtin feature, specify the "disable_post_builtins" parameter as follows:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "disable_post_builtins": true,
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

In this case, even though the docker command may create a HySDS dataset in the job's working directory, the `hysds.utils.publish_urls` post-processor will not run. To specify additional post-processor functions, define them using the "post" parameter like so:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "post": [ "my.custom.postprocessor_function" ],
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

Because the default builtin post-processor functions aren't disabled, the effect of the above `job-spec` is this:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "disable_pre_builtins": true,
  "post": [ "hysds.utils.publish_datasets", "my.custom.preprocessor_function" ],
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

### triage
HySDS provides an additional builtin post-processor function for providing triage of failed jobs: `hysds.utils.triage`. It is not enabled by default and must be explicitly set:

```
{
  "command":"/home/ops/ariamh/ariaml/extractFeatures.sh",
  "disk_usage":"1GB",
  "imported_worker_files": {
    "/home/ops/.netrc": "/home/ops/.netrc",
    "/home/ops/.aws": "/home/ops/.aws",
    "/home/ops/ariamh/conf/settings.conf": "/home/ops/ariamh/conf/settings.conf"
  },
  "post": [ "hysds.utils.triage" ],
  "params" : [
    {
      "name": "uri",
      "destination": "positional"
    },
    {
      "name": "uri",
      "destination": "localize"
    }
  ]
}
```

The above `job-spec` results in 2 post-processing functions being called after the docker command for the job executes: `hysds.utils.publish_datasets` and `hysds.utils.triage`. In both cases, the functions perform the proper checks up-front before continuing on with their functionality. For example, `hysds.utils.publish_datasets` checks the exit code of the docker command by inspecting the job dict and only proceeds with searching for HySDS datasets if the exit code was `0`. For `hysds.utils.triage`, the function checks that the exit code was not a `0` and continues with the creation and publishing of the triage dataset if so. If the job was triaged, there will be a `_triaged.json` file left in the job's work directory which contains the JSON result returned by the GRQ ingest REST call.

#### What gets triaged?
By default, all `_*` and `*.log` files in the root of the job work directory are triaged.

#### Can we triage other files as well?
The behavior of the triage function can be modified by updating certain fields in the `_context.json` of the job's work directory. Within the docker command that is called, the process can open up the `_context.json` and add a top-level parameter named `_triage_additional_globs` as a list of glob patterns that will be triaged in addition to the default files. For example, a python script that is called as a docker command for the job can add these files for triage:

```
with open(ctx_file) as f:
ctx = json.load(f)

ctx['_triage_additional_globs'] = [ 'S1-IFG*', 'AOI_*', 'celeryconfig.py', 'datasets.json' ]

with open(ctx_file, 'w') as f:
    json.dump(ctx, f, sort_keys=True, indent=2)
```

#### Triage is enabled in the `job-spec`. Can we disable triage at runtime?
Yes. Similar to the addition of other files to triage, you can add a top-level parameter named `_triage_disabled` to disable triage:

```
with open(ctx_file) as f:
ctx = json.load(f)

ctx['_triage_additional_globs'] = [ 'S1-IFG*', 'AOI_*', 'celeryconfig.py', 'datasets.json' ]

if some_condition:
    ctx['_triage_disabled'] = True

with open(ctx_file, 'w') as f:
    json.dump(ctx, f, sort_keys=True, indent=2)
```
