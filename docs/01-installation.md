---
title: Installing Polynote
layout: docs
---

# Installing Polynote

> ## Warning!
> Polynote is a web-based programming notebook tool. Like other notebook tools, a large part of its usefulness relies on
> **arbitrary remote code execution.**
>
> It currently contains **no built-in security or authentication** of its own, and relies entirely on the user deploying
> and configuring it in a secure way.
> 
> Polynote should only be deployed on a secure server which has its own security and authentication mechanisms which
> prevent all unauthorized network access.
>
> You are solely responsible for any damage or other loss that occurs as a result of running Polynote.
{:.warning} 

## License

Polynote is available under the [Apache License, Version 2.0](https://github.com/polynote/polynote/blob/master/LICENSE).

## Download
Polynote consists of a JVM-based server application, which serves a web-based client. To try it locally, find the latest
release on the [releases page](https://github.com/polynote/polynote/releases){:target="_blank"} and download the attached
`polynote-dist.tar.gz` file. Unpack the archive:

```
tar -zxvpf polynote-dist.tar.gz
cd polynote
```

## Prerequisites
- Polynote is currently only tested on Linux and MacOS, using the Chrome browser as a client. We hope to be testing
  other platforms and browsers soon. Feel free to try it on your platform, and be sure to let us know about any issues
  you encounter by filing a bug report.
- *Python support*: In order to execute Python code, you'll need to have Python 3 and pip3 installed. Refer to
  [Python's installation instructions](https://wiki.python.org/moin/BeginnersGuide/Download){:target="_blank"} for
  instructions on installing these packages.
  
  You'll also need to install some Python dependencies `jep`, `jedi`, `virtualenv` and `pyspark` (if using Spark). For example,
  
  ```
  pip3 install jep jedi pyspark virtualenv
  ``` 
- *Spark support*: In order to use Spark with kernel isolation, you'll need to [install Apache Spark&trade;](https://spark.apache.org/downloads.html).
  Polynote will use the `spark-submit` command in order to start isolated kernels.
  
- *Java 8* is currently required. Polynote does not currently run on later versions.  

## Configure
To change any of the default configuration, you'll need to copy the included `config-template.yaml` file to `config.yaml`.

## Run
To start the server, run the included bash script:

```
./polynote
```

If you don't have bash, or prefer to directly run the server, you can 

## Next
Once the server has started, navigate your browser to `http://localhost:8192` (if using the default network configuration).
Next, read about [basic usage](02-basic-usage.md)
