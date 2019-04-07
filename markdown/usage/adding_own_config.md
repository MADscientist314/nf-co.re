---
title: Adding your own cluster configuration
subtitle: How to handle your own specific configuration for other clusters
---

It is entirely possible to nf-core pipelines on other clusters, though you will need to set up your own config file so that the pipeline knows how to work with your cluster.

> If you think that there are other people using the pipeline who would benefit from your configuration (eg. other common cluster setups), please let us know. We can add a new configuration and profile which can used by specifying `-profile <name>` when running the pipeline. The config file will then be hosted at `nf-core/configs` and will be pulled automatically before the pipeline is executed.

If you are the only person to be running this pipeline, you can create your config file as `~/.nextflow/config` and it will be applied every time you run Nextflow. Alternatively, save the file anywhere and reference it when running the pipeline with `-c path/to/config` (see the [Nextflow documentation](https://www.nextflow.io/docs/latest/config.html) for more).

A basic configuration comes with all nf-core pipelines. This means that you only need to configure the specifics for your system and overwrite any defaults that you intend to change.

## Cluster Environment
By default, pipeline uses the `local` Nextflow executor - in other words, all jobs are run in the login session. If you're using a simple server, this may be fine. If you're using a compute cluster, this is bad as all jobs will run on the head node.

To specify your cluster environment, add the following line to your config file:

```nextflow
process.executor = 'YOUR_SYSTEM_TYPE'
```

Many different cluster types are supported by Nextflow. For more information, please see the [Nextflow documentation](https://www.nextflow.io/docs/latest/executor.html).

Note that you may need to specify cluster options, such as a project or queue. To do so, use the `clusterOptions` config option:

```nextflow
process {
  executor = 'SLURM'
  clusterOptions = '-A myproject'
}
```
## Software Requirements
To run your selected nf-core pipeline, several software packages are required. How you satisfy these requirements is essentially up to you and depends on your system. If possible, we _highly_ recommend using either Docker or Singularity.

Please see the [`installation documentation`](usage/installation) for how to run using the below as a one-off. These instructions are about configuring a config file for repeated use.

### Docker
Docker is a great way to run nf-core pipelines, as it manages all software installations and allows the pipeline to be run in an identical software environment across a range of systems.

Nextflow has [excellent integration](https://www.nextflow.io/docs/latest/docker.html) with Docker, and beyond installing the two tools, not much else is required - nextflow will automatically fetch the Docker image that we have created and is hosted at DockerHub at run time.

To add docker support to your own config file, add the following:

```nextflow
docker.enabled = true
```

> Note that the DockerHub organisation name annoyingly can't have a hyphen, so it's `nfcore` and not `nf-core`.

### Singularity image
Many HPC environments are not able to run Docker due to security issues.
[Singularity](http://singularity.lbl.gov/) is a tool designed to run on such HPC systems which is very similar to Docker.

To specify singularity usage in your pipeline config file, add the following:

```nextflow
singularity.enabled = true
```

If you intend to run the pipeline offline, nextflow will not be able to automatically download the singularity image for you. Instead, you'll have to do this yourself manually first, transfer the image file and then point to that.

First, pull the image file where you have an internet connection:

```bash
singularity pull --name nfcore-YOURPIPELINENAME-VERSION.simg nfcore/YOURPIPELINENAME:VERSION
```

Then transfer this file and point the config file to the image:

```nextflow
singularity.enabled = true
process.container = "/path/to/nfcore-YOURPIPELINENAME-VERSION.simg"
```
### Conda
If you're not able to use Docker or Singularity, you can instead use conda to manage the software requirements.
To use conda in your own config file, add the following:

```nextflow
process.conda = "$baseDir/environment.yml"
```