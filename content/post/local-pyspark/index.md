
+++
author = "Jose Torrado"
title = "Setting up a LocalDev Pyspark Environment with JupyterLab"
date = "2024-07-12"
description = "Easy setup to start learning PySpark"
tags = [
    "data-engineering",
    "distribute-compute",
    "tools"
]
categories = [
    "pyspark",
    "jupyter",
    "data-engineering"
]
image = "jupyter-spark.png"
+++

Spark is a staple in a data engineering toolset for distributed compute transformations. It does however have a level of overhead and dependencies that need to be sorted in order to set up. These are usually handled by Cloud Services natively (e.g. GCP Dataproc)

But if you are just getting started with learning Pyspark - 
or just want a lightweight and fast way to test some sytax locally, 
spinning up a cluster is additional complexity you might not want to deal with 
(not to mention it cost money)

A lot of the tutorials out there are not too intuitve, or are missing some key aspects
that make the development experince easier.
So I decided to compile a list of steps to get you started with pyspark locally quickly.
Hope you find it useful!



## Prereqs
- Python (will be using verison 3.10) 

Check if you have python by running 

``` bash
python --version
```

## Install Java

Check if you already have it installed

``` bash
java --version
```

If not, go [here](https://www.java.com/en/download/) and follow the installation instructions

## Install Spark

The following commands are for Mac (Bash) - if you are running a bash shell in Windows these should work. Otherwise, they _might_ need to be slightly altered to run in powershell

1. Go to the [Spark Download](https://spark.apache.org/downloads.html) page and get the latest version of spark

2. Unzip the downloaded `.tgz` file (zipped file)

3. Move the file to your `/opt/` folder in Mac:

``` bash
sudo mv spark-3.5.1-bin-hadoop3 /opt/spark-3.5.1
```

In Windows you might not have this folder.
The name or location is not really important as long as you have it somewhere it can't be
accidentally moved or deleted since it will be reference any time you spin up a notebook

4. Create a symbolic link to make it easier to switch Spark versions

If you want to install a different version of Spark in the future, you can just adjust the symlink and everything else will still work...

``` bash
sudo ln -s /opt/spark-3.5.1 /opt/spark
```

## Install Pyspark and Jupyter

To keep things clean it is better to always use virtual environments when installing python modules. This can be done in many ways: conda envs, pipenv, venv, ...

Here I will be demonstrating with `venv`

1. Create a new directory to work from: `mkdir jupyter-spark`
2. `cd jupyter-spark`
3. Create a virtual environment: `python -m venv .pyspark-en`
4. Activate the virtual environment: `source .pyspark-env/bin/activate`
5. Install pyspark and jupyterlab: `pip install pyspark jupyterlab`

## Updating your shell rc file

Now you need to tell your shell where to find Spark - this is done by setting some environment variables

This can be done a variety of ways, but to make it as seamless as possible we will be handling it in the `~/.bashrc`

_Note: depending on your shell, this step will be different. For example, if you use zsh as your shell, you will need to modify your `~/.zshrc` instead_

Add the following environment variables:
``` bash
# ~~~~~~~~~~~~~~~ Spark ~~~~~~~~~~~~~~~~~~~~~~~~
# Spark
export SPARK_HOME="/opt/spark"
export PATH=$PATH:$SPARK_HOME/bin
export SPARK_LOCAL_IP="127.0.0.1"

# Pyspark
export PYSPARK_DRIVER_PYTHON=jupyter
export PYSPARK_DRIVER_PYTHON_OPTS='lab'
export PYSPARK_PYTHON=python3
```

Save, exit and restart your shell

## Create a Pyspark Notebook

Done with the setup, now anytime you want to start your pyspark instance with JupyterLab locally you just need to:
1. cd into your directory where you installed pyspark
2. activate the virtual environment
3. run the command `pyspark` in your shell

This will open a JupyterLab instance in your default browser, and you are good to go!

