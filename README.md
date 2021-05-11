# PharmaDB

The PharmaDB project is a natural language processing data project used to associate changes in patent claims with changes in labels on drug patents, and to present these changes in a usable way in order give better insight into patent strategies and to better position a firm's existing patent strategies. This repository contains the instructions needed in order to run the PharmaDB project.


## Running PharmaDB

The core PharmaDB program is broken up into the front-end infrastructure and back-end infrastructure. The infrastructures are built separately and hosted using separate resources for improved isolation and security. Below are the instructions for setting up and running the front-end and back-end infrastructures.


### Running the PharmaDB Front-End Infrastructure

Below are the instructions for setting up and running the PharmaDB front-end infrastructure.


#### Installing the Single Page Application Dependencies

In order to build the single page application that the user's browser will download when running the program, the following dependencies need to be installed:

```
Git    : 2+
NodeJS : 10+
NPM    : 5+
```

However, older versions of the dependencies may work, but have not been tested.


#### Building the Single Page Application

In order to build the PharmaDB single page application, the following repositories need to be cloned:

```
capstone_spa (git clone https://github.com/pharmaDB/capstone_spa.git)
```

Once the repository has been cloned, the following commands can be used to build the single page application:

```
# While in the directory ~/capstone_spa/
npm install
ng build --prod
```

Once the build has completed, the build artifacts will be stored in the `~/capstone_spa/dist/` directory. 


#### Deploying the Single Page Application

Once the single page application has been built, the build artifacts can be hosted on any standard web server. If using AWS, it is recommend to store the single page application on an AWS S3 bucket with read access to only a specified list of internal IP addresses to serve up the application to user's browser. Note that CORS may need to be enabled and updated depending on the domains that are hosting the resources.


#### Setting Up the Front-End Web API Virtual Machine Instances

The built web API requires a virtual machine instance that may run one of the following operating systems:

```
Ubuntu 18.04
Ubuntu 20.04 
```

However, other operating systems and other versions may work as well, but have not been tested. If running the infrastructure on AWS, the following Amazon Machine Images (AMI's) may be used:

```
ami-042e8287309f5df03
```

However, other AMI's may work as well, but have not been tested.


#### Setting Up the Inbound and Outbound Traffic

The virtual machine instance requires inbound traffic for the following ports:

```
22 (SSH administration)
```

and the virtual machine requires outbound traffic for the following ports:

```
80 (HTTP protocol)
443 (HTTPS protocol)
```

If running the infrastructure on AWS, the inbound and outbound ports should be specified using an AWS Security Group.


#### Installing the Web API Dependencies

In order to build the web API that the single page application will interact with, the following dependencies need to be installed:

```
Git    : 2+
NodeJS : 10+
NPM    : 5+
```

However, older versions of the dependencies may work, but have not been tested.


#### Building the Front-End Web API

In order to build the PharmaDB web API on the virtual machine instance, the following repositories need to be cloned:

```
pharmadb_api (git clone https://github.com/pharmaDB/pharmadb_api.git)
```

Once the repository has been cloned, the following commands can be used to build the web API:

```
# While in the directory ~/pharmadb_api/

# Installing the NodeJS dependencies
npm install

# Note that you need to be running the web API locally before running this
# command in order for the web API to bundle properly
npm run webpack
```

Once the build has completed, the web API can be run using the follow command:

```
npm run start
```

You should then see the following text returned, which will tell you what port the server is listening on:

```
{"message":"Listening on port 3000","level":"info"}
```


### Running the PharmaDB Back-End Infrastructure

Below are the instructions for setting up and running the PharmaDB back-end infrastructure.


#### Setting Up the Virtual Machine Instances

The back-end infrastructure for PharmaDB requires a virtual machine instance that may run one of the following operating systems:

```
Ubuntu 18.04
Ubuntu 20.04 
```

However, other operating systems and other versions may work as well, but have not been tested. If running the infrastructure on AWS, the following Amazon Machine Images (AMI's) may be used:

```
ami-042e8287309f5df03
```

However, other AMI's may work as well, but have not been tested.


#### Setting Up the Inbound and Outbound Traffic

The virtual machine instance requires inbound traffic for the following ports:

```
22 (SSH administration)
```

and the virtual machine requires outbound traffic for the following ports:

```
80    (HTTP web hosting for the .csv export of the database)
27017 (MongoDB database)
27018 (MongoDB database)
27019 (MongoDB database)
8081  (Mongo Express administrative tool)
```

If running the infrastructure on AWS, the inbound and outbound ports should be specified using an AWS Security Group.


#### Setting Up the Virtual Machine Storage

The provisioned virtual machine instance should also have additional storage capacity to store all of the data in the MongoDB instance. The storage space requirements are as follows:

```
Minimum     : 128 GB
Recommended : 256 GB
```

The NLP engine can take advantage of the GPU automatically if one is available.  Otherwise, it is still able to run on a server with just 4GB of ram, albeit slowly.


#### Installing the MongoDB Dependencies

Once the virtual machine instance has been installed and is running, the PharmaDB back-end applications can be installed. First, the MongoDB database should be installed and running. In order to install the MongoDB database, the following dependencies are required:

```
Docker           : 20+
Docker Compose   : 1.29+
Docker Engine CE : 19+
```

However, older versions of the dependencies may work, but have not been tested.


#### Installing and Running MongoDB

Once the Docker dependencies are installed, a MongoDB container from Docker Hub can be installed in order to get the MongoDB up and running. The following command may be used to pull the docker container:


```
sudo docker pull mongo
```

and the following command can be used to run the MongoDB container:

```
sudo docker run \
    -p 27017-27019:27017-27019 \
    --name mongodb \
    -d mongo:4.4
```


#### Installing the Back-End Application Dependencies

After the MongoDB is up and running, the back-end applications need to be downloaded and run. The following are the dependencies needed to run all of the back-end applications used to build the database and run the ETL pipeline:

```
Git    : 2+
Python : 3+
NodeJS : 10+
NPM    : 5+
```

However, older versions of the dependencies may work, but have not been tested.

Once the dependencies are installed, all of the back-end application repositories need to be cloned from the git repositories. The following repositories need to be cloned:

```
etl_pipeline                 (git clone https://github.com/pharmaDB/etl_pipeline.git)
scoring_data_processor       (git clone https://github.com/pharmaDB/scoring_data_processor.git)
dailymed_data_processor      (git clone https://github.com/pharmaDB/dailymed_data_processor.git)
data_analysis                (git clone https://github.com/pharmaDB/data_analysis.git)
uspto_bulk_file_processor_v4 (git clone https://github.com/pharmaDB/uspto_bulk_file_processor_v4.git)
```

Additional libraries for each repository need to be installed. The following commands will install the libraries for each of the cloned repositories:

**etl_pipeline**

```
# While in the directory ~/etl_pipeline/
pip3 install -r requirements.txt
```

**scoring_data_processor**

```
# While in the directory ~/scoring_data_processor/
pip3 install -r requirements.txt
```

**dailymed_data_processor**

```
# While in the directory ~/dailymed_data_processor/
pip3 install -r requirements.txt
```

**uspto_bulk_file_processor_v4**

```
# While in the directory ~/uspto_bulk_file_processor_v4/
npm install
```

Additionally, some repositories require additional build steps. The following commands can be used to build the programs in the repositories:

**uspto_bulk_file_processor_v4**

```
# While in the directory ~/uspto_bulk_file_processor_v4/
npm run build
```


#### Running the Back-End Applications

After cloning the back-end applications and installing all of their dependencies, the back-end applications need to be run in order to build all of the data in the database. The following commands for each of the repositories will build the database. Note that the MongoDB database needs to be running and that the commands should be run in the order provided below in order for the applications to run properly:

**etl_pipeline**

```
python3 -m ./etl_pipeline/main.py
```

**dailymed_data_processor**

```
python3 -m ./dailymed_data_processor/main.py
```

**uspto_bulk_file_processor_v4**

```
node ./uspto_bulk_file_processor_v4/out/index.js \
    --patent-number-file "patents.json"
```

**scoring_data_processor**

```
python3 ./scoring_data_processor/main.py
```

#### Setting Up the Recurring Monthly Data Refreshes

The above commands will build the static MongoDB database using all available data when run. As new data is added to the various data sources that the PharmaDB project is pulling from (such as the DailyMed and USPTO), monthly refresh scripts can be used to keep the MongoDB database up to date with this new data. The following commands can be added to a recurring cron job program to refresh the data in the database with new data. The below commands are set up to refresh data once a month, but more frequent refreshes can be run using different parameters:

**etl_pipeline**

```
python3 -m ./etl_pipeline/main.py
```

**dailymed_data_processor**

```
python3 -m ./dailymed_data_processor/main.py
```

**uspto_bulk_file_processor_v4**

```
node ~/uspto_bulk_file_processor_v4/out/index.js \
    --patent-number-file "patents.json" \
    --start-date "$(date +"%Y-%m-01" -d "-1 month")" \
    --end-date "$(date +"%Y-%m-01")"
```

**scoring_data_processor**

```
python3 ./scoring_data_processor/main.py
```