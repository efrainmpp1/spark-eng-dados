# spark-eng-dados

## Spark and Hadoop Configuration and Release Information ##

Spark Version `2.4.5`, Hadoop version is `2.7`, Python version `3.7.3`.

Apache Spark is running in *Standalone Mode* and controls its own master and worker nodes instead of Yarn managing them.     

Apache Spark with Apache Hadoop support is used to allow the cluster to simulate HDFS distributed filesystem using the shared volume `shared-workspace`.

# Build ##
## Quick-Start ##

Ensure that the Docker environment has enough memory allocated:
- Configure a minimum of 4GB in Docker Resources, ideally 8GB   


Build the images with
 ```
 build.sh
 ```     
Create the Docker volumes before starting services:
 ```
 docker volume create --name=hadoop-distributed-file-system
 ```  
Start the cluster with:  
```
docker-compose up --detach
```

Test the cluster using notebook `./local/notebooks/search_comapnies.ipynb`  
- Use the JupyterLab environment which should now be available on http://localhost:8889/

## Build Overview ##


The following Docker images are created:  
+ `cluster-base` - this provides the shared directory (`/opt/workspace`) for the HDFS simulation.  
+ `spark-base`  - base Apache Spark image to build the Spark Master and Spark Workers on.   
+ `spark-master` - Spark Master that allows Worker nodes to connect via SPARK_MASTER_PORT, also exposes the Spark Master UI web-page (port 8080).  
+ `spark-worker` - multiple Spark Worker containers can be started from this image to form the cluster.    
+ `jupyterlab` -  built on top of the cluster-base with Python and JupyterLab environment with an additional filesystem for storing Jupyter Notebooks and spark-submit scripts.

The cluster is dependent on a shared volume `shared-workspace` that is created during the docker-compose initialisation
```
volumes:
  shared-workspace:
    name: "hadoop-distributed-file-system"
    driver: local
```

Once created, the data in shared-workspace is persistent in the Docker environment.

## Cluster Dependencies ##

*Docker Compose* is used to link all the cluster components together so that an overall running cluster service can be started.  

`docker-compose.yml` initialises a shared cluster volume for the shared filesystem (HDFS simulation) and also maps `./local/notebooks` to a mount point in the JupyterLab Docker container.  

Various other port-mappings and configuration details are set in this configuration file.  Because all the worker nodes need to be referenced at `localhost`, they are mapped to different port numbers (ports 8081 and 8082 for worker 1 and 2).

## Start Cluster ##

```
docker-compose up --detach
```

## Stop Cluster ##
```
docker-compose down
```
## Shared JupyterLab Notebooks Folder ##

The folder `./local/notebooks` in the Git repo is mapped to a filesytem mount `/opt/workspace/notebooks` in the JupyterLab host running in Docker.  This allows notebooks to be easily passed in and out of the Docker environment, and they can be stored and updated independently of the images.  

This folder is included in the `.gitignore` file so updates to the notebooks and new notebooks are not tracked.  


## Data Folders ##

Three shared cluster-wide folders are created on top of `/opt/workspace` during the Docker build:
+ `/opt/workspace/events` - Spark history events  
+ `/opt/workspace/datain` - source data for loading in to Spark jobs  
+ `/opt/workspace/dataout`- output data from Spark jobs  

The data in these folders is persistent between container restarts and between Docker image rebuilds as it is located on the Docker `shared-workspace` volume.

#### Download Sample Data ####

Sample data for the notebooks can be downloaded by website provides a dedicated section where users can easily access and download sample data specifically curated for use in the notebooks. For the most up-to-date and relevant sample datasets, please visit [websiteURL](https://dados.gov.br/dados/conjuntos-dados/cadastro-nacional-da-pessoa-juridica---cnpj).