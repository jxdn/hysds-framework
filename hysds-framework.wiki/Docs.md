This section provides documentation on the core HySDS system.

# General HySDS Terminology
| term | description |
| ---- | ----------- |
| `PGE` | Product Generating Executor - a set of executable binaries and/or scripts that generate an output file or files of interest |
| `non-PGE` | a set of executable binaries and/or scripts that perform some action of which their output files are inconsequential |
| `job` | a versionized PGE/non-PGE or instance thereof that is wrapped and encapsulated in a docker container; within the container, PGEs are wrapped to generate datasets |
| `job-spec` | HySDS job specification; usually refers to the JSON file that contains it `job-spec.json` |
| `hysds-io` | HySDS I/O specification; usually refers to the JSON file that contains it, `hysds-io.json` |
| `worker` | a celery worker that executes jobs |
| `worker node` | a compute instance that runs one or many workers; synonymous with `verdi` node or `verdi` instance |
| `dataset` | a single, logical product type or instance thereof that encapsulates all files of interest generated by a PGE into a directory along with inherent HySDS-specific JSON files |
| `wrapper` | an executable binary or script that executes a PGE to generate an output file or files of interest, forms a dataset, and may execute other non-PGE tasks |
| `cluster` | an instantiation of a group of HySDS component instances composed of (at a minimum) `mozart`, `metrics`, `grq`, `factotum`, and `ci` instances |
| `mozart` | the HySDS component that orchestrates jobs, job queues, and other resources |
| `metrics` | the HySDS component that provides job and worker node metrics |
| `grq` | the HySDS component that provides the dataset catalog, APIs manipulating them, and various search interfaces  |
| `verdi` | the HySDS component that runs jobs; may be referred to as `worker node` or `worker instance` |
| `factotum` | the HySDS component that is a specialized `verdi` instance for processing rule triggers and on-demand requests; may also be used for one-off tasks such as connection-throttled data downloads or data source API querying |
| `ci` | the HySDS component that provides continuous integration of versioned PGE/non-PGE containers and automates integration into the cluster  |

# Setup
- [[Installation]]
- [[Cluster Setup]]
- [[Hello World]]
- [[Hello Dataset]]
- [Datasets Reference](Datasets)
- [[Puppet Automation]]
- [[FAQs]]
- [[How Tos]]

# HySDS Core
- [[Resource Management]] (Mozart)
- [[Dataset Search]] (Tosca)
- [[Metrics]]
- [[System Light-Weight Jobs]] (Factotum)
- [[Workers]] (Verdi)
- [[Workflows]]
