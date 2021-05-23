# PharmaDB

The PharmaDB project is a natural language processing data project used to associate changes in patent claims with changes in labels on drug patents, and to present these changes in a usable way in order give relevant insight into patent strategies and to better position a firm's existing patent strategies. This repository contains the instructions needed to run the PharmaDB project.


## Running PharmaDB

The core PharmaDB program is broken up into the front-end infrastructure and back-end infrastructure. The infrastructures are built separately and hosted using separate resources for improved isolation and security. Below are the instructions for setting up and running the front-end and back-end infrastructures.


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
8081  (Mongo Express administrative tool)
```

If running the infrastructure on AWS, the inbound and outbound ports should be specified using an AWS Security Group.

**Please note the address and port of the MongoDB database of virtual machine for the Back-End Infrastructure.  The address needs to be set in various modules to complete the installation process.  If the database and front-end application are run on localhost, the address and port are `localhost` and `27017`, respectively.  If the application is hosted using a cloud computing service, the address is the address of the cloud computing service.  An example address and port are `ec2-35-170-81-159.compute-1.amazonaws.com` and `27017`, respectively.**


#### Setting Up the Virtual Machine Storage

The provisioned virtual machine instance should also have additional storage capacity to store all of the data in the MongoDB instance. The storage space requirements are as follows:

```
Minimum     : 2 TB
Recommended : 3 TB
```

The NLP engine can take advantage of the GPU automatically if one is available.  Otherwise, it is still able to run on a server with just 4GB of ram, albeit slowly.


#### Installing the Back-End Application Dependencies

Once the virtual machine instance has been installed and is running, the PharmaDB back-end applications can be installed. The following dependencies are required to install the Back-End Application:

```
Docker           : 20+
Docker Compose   : 1.29+
Docker Engine CE : 19+
Git              : 2+
Python           : 3.7+
NodeJS           : 10  <-- uspto-patent-processor will not run with NodeJS version 16
NPM              : 5+
```

Older versions of the dependencies may work, but have not been tested.


#### Installing the Back-End Application Dependencies

Once all dependencies are met, we need to set up the [ETL pipeline](https://github.com/pharmaDB/etl_pipeline).


Clone this repo, with the submodules
```
$ git clone --recurse-submodules https://github.com/pharmaDB/etl_pipeline.git
```
Build the `node` project for the patent data collection.
```
$ cd src/submodules/uspto_bulk_file_processor_v4
$ npm install
$ npm run build
$ cd -
```
Install the Python project dependencies in the submodules.
```
$ cd src/submodules/dailymed_data_processor
$ pip3 install -r requirements.txt
$ cd -
```
```
$ cd src/submodules/scoring_data_processor
$ pip3 install -r requirements.txt
$ cd -
```

#### Setup .env files

In **Setting Up the Inbound and Outbound Traffic**, it was advised to track the address and port of the MongoDB server.  This information should now be placed in respective `.env` files.

The location of the `.env` files in `etl_pipeline` repository are as follows:

```
etl_pipeline/src/.env
etl_pipeline/src/submodules/dailymed_data_processor/.env
etl_pipeline/src/submodules/scoring_data_processor/.env
```

Ensure that :

```
MONGODB_HOST=localhost
MONGODB_PORT=27017
MONGODB_NAME=pharmadb
```

`localhost` should be changed to address of the MongoDB server if it is not hosted locally with respect to the downloaded `etl_pipeline` repository.


#### Start Servers

Start the file server (for the CSV export download) in the background, from the scoring submodule.  This file-server hosts the export of the labels collection in MongoDB.

```
sudo nohup python3 src/submodules/scoring_data_processor/server.py &
```

Build the docker image using:

```
docker-compose build
```

Start the container using:

```
docker-compose up
```

`docker ps -a` should now indicate that `mongo_local` is the name of the docker container that is running MongoDB on port `27017`.


#### Load Data to Server

Unzip all pre-process database collections and related files:

```
(go back to the root folder for etl_pipeline/)
cd src/submodules/scoring_data_processor/resources/
tar -xf database_latest.tar.gz
tar -xf processed_log.tar.gz
cd -
```

The label and patent data for older years has been gathered using a combination of sources and clean up tasks (some of these are captured in Jupyter notebooks in the [data_analysis](https://github.com/pharmaDB/data_analysis) repository). To import the bulk data and scores, correct as of May 2021, please use the following argument:

```
python3 src/submodules/scoring_data_processor/main.py -ril -rilm -rio
```

To load patent data onto the database, use the following command.  (Warning: This command may take a long time to complete, since it is downloading the entire US patent database from 1985 to present.  This command also need ~1.5TB of storage space):

```
echo "[]" > empty_array.json && node out/index.js --connection-string="mongodb://localhost:27017/pharmadb" --patent-number-file="empty_array.json"
```


#### Run Periodic data collection and ML scoring

To update the database, which should occur monthly, use:

```
python3 src/main.py
```

Upon successful pipeline run, the data in MongoDB should be updated. As a quick check, the `pipeline` collection should show the updated timestamp for the latest successful run. The CSV export of the DB should also be updated in `src/submodules/scoring_data_processor/resources/hosted_folder/db2csv.zip`.  The Front End is able to retrieve this file and make it available for downloading, to the users. For supporting this functionality, the exported data must be hosted using a server, which was started in **Start Servers**.

The script can also be run monthly, as a cron job. To setup a cronjob, run:

```
export VISUAL=vi
crontab -e
```

Then enter:
```
0 0 1 * * python3 main.py
```

Followed by pressing ESCAPE and typing:
```
:wq<ENTER>
```


### Running the PharmaDB Front-End Infrastructure

#### Installing the Single Page Application Dependencies

In order to build the single page application that the user's browser will download when running the program, the following dependencies need to be installed:

```
Git    : 2+
NodeJS : 10+
NPM    : 5+
```

** Older versions of the dependencies may work, but have not been tested.


#### Building the Single Page Application

In order to build the PharmaDB single page application, the following repositories need to be cloned:

```
capstone_spa (git clone https://github.com/pharmaDB/capstone_spa.git)
```

Once the repository has been cloned, please proceed to change `href="http://.../db2csv.csv.zip"` in line 15 of  `src/app/shared/navbar/navbar.component.html` to point to the address of the MongoDB database that was previously set-up in **Setting Up the Inbound and Outbound Traffic** of the **PharmaDB Back-end Infrastructure** above.  An example address is shown below:

```
href="http://ec2-35-170-81-159.compute-1.amazonaws.com/db2csv.csv.zip"
```
or

```
href="http://localhost/db2csv.csv.zip"
```

Next, the following commands can be used to build the single page application:

```
# While in the directory ~/capstone_spa/
npm install
ng build --prod
```

Once the build has completed, the build artifacts will be stored in the `~/capstone_spa/dist/` directory. 


#### Deploying the Single Page Application

Once the single page application has been built, the build artifacts can be hosted on any standard web server. If using AWS, it is recommend to store the single page application on an AWS S3 bucket with read access to only a specified list of internal IP addresses to serve up the application to user's browser. Note that CORS may need to be enabled and updated depending on the domains that are hosting the resources.


#### Setting Up the Web API Virtual Machine Instances

The built web API requires a virtual machine instance that may run one of the following operating systems:

```
Ubuntu 18.04
Ubuntu 20.04 
```
** Other operating systems and other versions may work as well, but have not been tested.

If running the infrastructure on AWS, the following Amazon Machine Images (AMI's) may be used:

```
ami-042e8287309f5df03
```
** Other AMI's may work as well, but have not been tested.


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

Once the repository has been cloned, please proceed to change `uri=mongodb://...` in line 14 of  `pharmadb_api/src/services/Mongo.services.ts` to point to the address and port of the MongoDB database that was previously set-up in **Setting Up the Inbound and Outbound Traffic** of the **PharmaDB Back-end Infrastructure** above.  An example address and port is shown below:

```
uri = 'mongodb://ec2-35-170-81-159.compute-1.amazonaws.com:27017'
```
or

```
uri = 'mongodb://localhost:27017'
```


Next, the following commands can be used to build the web API:

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


