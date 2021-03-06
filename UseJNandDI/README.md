# SAP Data Intelligence and Jupyter Notebook
Currently it is still kind of awkward to use the Jupyter Notebook environment with SAP Data Intelligence. But we hope it will soon improve. In the meanwhile when you like to use data and data source you can use the following guideline. 

# Scenario Manager 
The scenario manager is the entry point that encompasses 

* Datasets - created with the Scenario manager
* Jupyter Notebooks
* Pipelines - created with the Scenario Manager

## Scenario Manager Pipelines
When you like to access DI data you need first copy it to the internal DI Data Lake that is a pre-defined connection to a *Semantic Data Lake (SDL)*. 

### Copied from SAP SDL Help: 
### Semantic Data Lake (SDL)


Allows connecting to a Semantic Data Lake. For Data Intelligence (cloud only), a connection, DI\_DATA\_LAKE, is pre-defined.

**Features** 

* Use in model training and model serving with Machine Learning scenarios

* Experiment with data using JupyterLab in the context of Machine Learning

* Read and write data in Modeler via Read File, Write File, and Flowagent File operators

* Explore content using Metadata Explorer's Browse Connections capability

* Obtain metadata and execute data profiling using Metadata Explorer

* Perform data operations in Modeler via Data Transform operator

![Connection Page](./images/Connections_SDL.png)

## Creating a Data Set Pipeline

When creating a data pipeline with the Scenario Manager it is managed by an internal *Git*. If you want to use an existing pipeline you need to create an empty pipeline from the Scenario Manager and then copy the JSON code to it. 

![Connection Page](./images/SM_CreatePipeline.png)

After the **Modeler** has been started you can create a simple pipeline that reads the data from a source/connection and writes it to the DI\_DATA\_LAKE. The following pipeline reads an *csv-file* from an object store. 

![Connection Page](./images/Pipeline.png)

In the configuration be a aware of the parameters

* *Artifact Kind*: dataset and 
* *Filename suffix*: .csv

After the completion of the pipeline a new **data** set is been created. 

![Connection Page](./images/SM_Dataset.png)

## Viewing a Data Set in Metadata Explorer

For viewing the dataset in the **Metadata Explorer** you need the **execution_id** (not to be confused with the *Technical Identifier* seen in the table). For example open the link of a dataset in a new browser tab: 

 ![link_dataset](./images/link_dataset.png)
  
and copy the link to the *Metadata Explorer*, connection: *DI\_DATA\_LAKE*.

![artifactlink](./images/artifact_link.png)

![dataset_metadata_explorer](./images/dataset_metadata_explorer.png)


## Accessing a Data Set in Jupyter Notebook
The *DI\_DATA\_LAKE* is a HDFS storage that you can access using the package **HdfsCLI** (https://hdfscli.readthedocs.io/en/latest/).

For loading and saveing datasets or other data you can use the following code snippet: 

```  python
from hdfs import InsecureClient
import os
import io

# dataset reading 
def get_dataset(exec_id,basepath ="/worm/sap/di/ml/artifacts/executions/",hdfs_url = 'http://datalake:50070' ) :
    logging.getLogger('hdfs').setLevel(logging.WARNING)
    client = InsecureClient(hdfs_url)
    
    artifact_folder_file = os.path.join(basepath,exec_id)
    folder_list = client.list(artifact_folder_file)
    artificat_dataset = os.path.join(artifact_folder_file,folder_list[0])
    
    # Loading a file in memory.
    with client.read(artificat_dataset) as reader:
        csv_file = reader.read()
    dataset = io.BytesIO(csv_file)
    
    return dataset

# saving data to hdfs
def hdfs_save(exec_id,filename,data,basepath =  "/worm/sap/di/ml/artifacts/executions/",hdfs_url = 'http://datalake:50070',\
             overwrite=False) :
    client = InsecureClient(hdfs_url)
    artifact_folder = os.path.join(basepath,exec_id)
    artificat_filename = os.path.join(artifact_folder,filename)
    client.write(artificat_filename,data,overwrite=overwrite)
    
# dataset saving 
def save_dataset(exec_id,filename,data,basepath =  "/worm/sap/di/ml/artifacts/executions/",hdfs_url = 'http://datalake:50070') :
    df_str = data.to_csv(sep=';',index=False)
    df_b = str.encode(df_str)
    hdfs_save(exec_id = exec_id,filename = filename, data = df_b, basepath = basepath, hdfs_url = hdfs_url)

```





