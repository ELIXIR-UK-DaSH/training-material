---
layout: tutorial_hands_on

title: "Working with Beacon V2: A Comprehensive Guide to Creating, Uploading, and Searching for Variants with Beacons"
zenodo_link: ''
subtopic: ''
questions:
- What does the term "beacon" refer to?
- How can MongoDB be employed to establish a beacon tailored for your institution?
- In what manner can variant data and metadata be readied into a format compatible with beacons?
- What are the steps involved in importing data into a beacon seamlessly?
- How does one perform queries on a beacon to retrieve information about variants?
objectives:
- Gain a comprehensive understanding of beacons by unveiling their concept and exploring their significance
- Master the skill of utilizing MongoDB to construct beacons, fostering institutional advancement through effective data management
- Learn the art of preparing data for beacons, including the transformation of variants and metadata into structures compatible with beacon requirements
- Achieve proficiency in seamlessly importing data into beacons through a step-by-step guide, ensuring a smooth and error-free process
- Develop the ability to query beacons for variants, unleashing the formidable capabilities of MongoDB in conducting powerful and precise variant searches
time_estimation: ?H
key_points:
- Understanding Beacons involves grasping their definition and concept, recognizing their significance in efficient data management, and exploring diverse use cases for these essential data structures.
- Mastering MongoDB for Beacon Creation entails familiarizing yourself with the MongoDB database system, acquiring the skills to construct beacons for advancing institutional data management,
  and exploring tailored MongoDB features for efficient beacon development.
- Effective Data Preparation for Beacons entails mastering techniques for organizing variant data, ensuring proper formatting of metadata to meet beacon requirements,
  and acknowledging the crucial role of well-prepared data in optimizing beacon functionality.
- Achieve seamless data importation into beacons by following a comprehensive step-by-step guide, prioritizing data integrity throughout the process,
  and acquiring skills in effective troubleshooting and error handling.
- Efficiently querying beacons for variants involves mastering diverse querying methods, leveraging MongoDB's capabilities to enhance search precision,
  and emphasizing the necessity for precise and efficient queries within beacon structures.
contributors:
- Khaled196
- kkamieniecka
- poterlowicz-lab
---


# Introduction

Beacon v2 is a data query protocol and API that allows the researcher to seek information about specific genomic variants of biomedical research and clinical applications from the data providers (beacon provider) without accessing the original or the whole dataset {% cite Rambla2022 %}. The protocol was developed by the Global Alliance for Genomics and Health (GA4GH) in 2021 as an update for the former Beacon v1 {% cite GA4GH %}{% cite ELIXIR Beacon %}. The second version of Beacon comes with additional features that provide users with more detailed information about the queried variants. Unlike the previous version, which only returned a Yes or No response indicating the presence of variants, Beacon v2 presents comprehensive results about the variants being searched. {% cite Rueda2022 %}.
The Beacon v2 comprises two main parts, namely the framework and the models. The framework specifies how the requests and responses should be formatted, while the models determine the organization of the biological data response {cite Rueda2022}.

![Beacon Framework and Model](../../images/Beacon_framework_and_model.png "Schematic representation of Beacon v2 API specification")

Before working with a beacon, the beacon providers must prepare the data in JSON format, also known as Beacon-friendly format (BFF), using the Beacon RI tools. The data is divided into two parts - Metadata and Genomic variants. The metadata is initially saved in different formats like Excel format, CSV, etc., while the Genomic variations are kept in VCF format. By converting the metadata into BFF format, it generates analyses, biosamples, cohorts, datasets, individuals, and runs, while the genomic variation VCF generates a genomicVariations file. When combined, they produce the seven entities present in the Beacon v2 Models (default schemas) and their relationships, as represented in the diagram below:

![Beacon v2 inputs](../../images/BBeacon_seven_inputs.png "Diagram representing the seven entities present in the Beacon v2 Models (default schemas) and their relationships")

The Beacon provider will utilize the Beacon ri tools to upload the necessary data into MongoDB after collecting it. However, in this tutorial, we will use distinct tools to import and retrieve data from Beacon v2, which was created by The University of Bradford Computational and Data-Driven Science research team.

Beacon MongoDB databases can be created with different security levels for accessing Beacon data through import and query processes. These databases can be classified into three security levels: public, registered, and controlled. Alternatively, they can be classified into two security levels in our import and query tools: public and authenticated.

|---------|---------|
|Sequrity Level|Description|
|Public|Beacon can be accessed by any request|
|Registered|Only known users can access the data|
|Controlled|Only specifically granted users can access the data|


In this tutorial, we will walk through the process of creating the Beacon v2 protocol using MongoDB. This will include showing how to prepare the data, import it and demonstrate how to perform queries.

> <agenda-title></agenda-title>
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}


# Create Beacon data discovery protocol using MongoDB

Beacon is created bu the Beacon providing instituion to serve and work as the institution prespective and if they want to make their beacon as open access or requers authentication to query the data. 

In this tutorial we will show an example on how to create both of those typs of beacon by creating 2 docker servers for MongoDB one for open access and the other Created from B2RI docker image.

## Create and open access MongoDB server

> <hands-on-title>Create open access Beacon Database on MongoDB</hands-on-title>
>
> 1. create directory on your local envirment and give it a suitble naming
>
> ```
> $mkdir <directory_name>
> ```
>
> 2. Move to the created Directory on the shell
> ```
> $cd <directory_name>
> ```
> 3. crate an empty file and name it 'docker-compose.yaml' using any text editor you have.
> ```
> $nano docker-compose.yaml
> ``` 
> 4. copy the text below into the 'docker-compose.yaml' file. 
> ```
> version: '3.6'
> services:
> 
>   mongo-client:
>     image: mongo:3.6
>     restart: unless-stopped
>     volumes:
>       - ./beacon/db:/data/db
>     ports:
>       - "27017:27017"
> 
>   mongo-express:
>     image: mongo-express
>     restart: unless-stopped
>     environment:
>       - ME_CONFIG_MONGODB_SERVER=mongo-client
>       - ME_CONFIG_MONGODB_PORT=27017
>       - ME_CONFIG_BASICAUTH_USERNAME=admin
>       - ME_CONFIG_BASICAUTH_PASSWORD=adminpass
>     ports:
>       - "8081:8081"
> ```
> 5. create the path 'beacon/db' in your directory using 'mkdir' tool
> ```
> $mkdir beacon
> $mkdir beacon/db
> ```
> You can change the name of that bath but you have to change that also from the docker-compose.yaml file. 
> Now we have everything ready for creating the MongoDB server hosted in docker container the only step left is to run docker. 
> 5. run the tool 'docker-compose' with the following parametars
> ```
> $docker-compose up -d
> ```
> 6. Check the created docker containers and test if your dokcer container is runinig
> ```
> $docker ps
> ```
> This will give you a massage similaer to this
> ```
> CONTAINER ID   IMAGE           COMMAND                  CREATED      STATUS          PORTS                                           NAMES
> 6a6d32af7952   mongo:3.6       "docker-entrypoint.s…"   8 days ago   Up 8 days       0.0.0.0:27017->27017/tcp, :::27017->27017/tcp   mongo-client
> ```
> 7. Test your dokcer to see if it runing by 
> ```
> $docker run -d -p 27017:27017 --name mongo-client mongo:3.6
> $docker exec -it mongo-client bash
> ```
> this take you into the docker container, you can exit that by prising 'ctrl + d' from your keyboard.  
>
> With this we have created an empty MongoDB server were we can add the beacon database or additional databases to it. 
{: .hands_on}


## Create authentecated beacon MongoDB database

> <hands-on-title>Create authentication requierd Beacon Database on MongoDB</hands-on-title>
>
> 1. create directory on your local envirment and give it a suitble naming
>
> ```
> $mkdir <directory_name>
> ```
>
> 2. Move to the created Directory on the shell
> ```
> $cd <directory_name>
> ```
> 3. crate an empty file and name it 'docker-compose.yaml' using any text editor you have.
> ```
> $nano docker-compose.yaml
> ``` 
> 4. copy the text below into the 'docker-compose.yaml' file. 
> ```
> version: '3.1'
> 
> # networks:
> #   beacon-priv:
> #   idp-priv:
> #   pub:
> 
> services:
> 
>   ###########################################
>   # MongoDB Database
>   ###########################################
> 
>   mongo:
>     image: mongo
>     hostname: mongo
>     ports:
>       - 27017:27017
>     environment:
>       MONGO_INITDB_ROOT_USERNAME: root
>       MONGO_INITDB_ROOT_PASSWORD: example
>     # networks:
>     #   - beacon-priv
> 
>   mongo-express:
>     image: mongo-express
>     restart: always
>     ports:
>       - 8081:8081
>      environment:
>       ME_CONFIG_MONGODB_ADMINUSERNAME: root
>       ME_CONFIG_MONGODB_ADMINPASSWORD: example
>       ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/
>     # networks:
>     #   - beacon-priv
> ```
> Now we have everything ready for creating the MongoDB server hosted in docker container the only step left is to run docker. 
> 5. run the tool 'docker-compose' with the following parametars
> ```
> $docker-compose up -d
> ```
> 6. Check the created docker containers and test if your dokcer container is runinig
> ```
> $docker ps
> ```
> This will give you a massage similaer to this
> ```
> 6a6d32af7952   mongo       "/sbin/tini -- /dock…"   8 days ago   Up 8 days       0.0.0.0:27017->27017/tcp, :::27017->27017/tcp   beacon_mongo_1
> 484aebfefcc6   mongo-express   "/sbin/tini -- /dock…"   8 days ago   Up 10 seconds   0.0.0.0:8081->8081/tcp, :::8081->8081/tcp       beacon_mongo-express_1
> ```
>
> With this we are using the Beacon MongoDB database templet created by the B2RI check the [github](https://github.com/EGA-archive/beacon2-ri-tools/tree/main) repo for more intormation
{: .hands_on}

# Import data into Beacon MongoDB

# Search Beacon MongoDB

# prepare the Data for MongoDB


# Conclusion

