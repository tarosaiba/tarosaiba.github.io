# Quickly set up a local development environment for Cloud Spanner with docker-compose
## Introduction

This article introduces the procedure to quickly set up a local development environment for Cloud Spanner using docker-compose. Recommended for those who develop with Cloud Spanner and for those who are curious about it and would like to try a few touches.

## What is Cloud Spanne?
Let me briefly introduce Cloud Spanner (referred to as "Spanner").

Spanner is the only relational database service that combines powerful consistency and horizontal scalability offered by Google Cloud.

The following is from the official documentation

> ・Get all the benefits of relational semantics and SQL with unlimited scale
> ・Start at any size and scale with no limits as your needs grow
> ・Enjoy high availability with zero scheduled downtime and online schema changes
> ・Deliver high-performance transactions with strong consistency across regions and continents
> ・Focus on innovation, eliminating manual tasks with capabilities like automatic sharding

It's a dreamy database service. But then again, it sounds expensive... The fee is as follows.

#### Price per node (including all replication) as of March 2021.3

| Type             | Region                 | $/hour | $/month (100%) |
|------------------|------------------------|--------|-----------------|
| Single-region    | asia-northeast1 (Tokyo) | 1.17  | 842.4           |
| Multi-region     | asia1 (Tokyo+Osaka)     | 3.9   | 2808            |

That's quite a lot of cost even for the smallest configuration, region + 1 node configuration.


## How do we prepare development environment
Because of the high cost, it seems difficult to casually set up an instance for a development environment.
Therefore, in this article, we will use [Spanner emulator](https://cloud.google.com/spanner/docs/emulator?hl=ja), which is officially provided by GCP, to set up a development environment! (Thank goodness for emulators!)

It is available in gcloud CLI and docker image, but we will show an example of using it with docker-compose.
Sa
Sample code is here: [**tarosaiba/docker-compose-spanner**](https://github.com/tarosaiba/docker-compose-spanner)


Two points of this code set.
* 通常、Spannerエミュレータ起動後にインスタンスの作成手順(`gcloud spanner instances create`)が必要になりますが、docker-compose立ち上げ時に自動でインスタンス作成されるようにしています
* ✔ Normally, the instance creation procedure (`gcloud spanner instances create`) is required after launching the Spanner emulator, but I have made docker-compose so that instances are created automatically when we execute `docker-compose up`.
* For DB initialization process (table creation & data submission), pre-prepared DDL/DML is automatically executed when docker-compose is started up.

So here are the steps.

## Requirement
* docker >= 19.03.0+
* docker-compose >= 1.27.0+

## Steps
### Quick start
* Clone sample repository [https://github.com/tarosaiba/compose-spanner](https://github.com/tarosaiba/compose-spanner)
* Change directry  `cd compoose-spanner`
* Launch docker-compose `docker-compose up -d`

That's all for the steps!

### How to connect to Spanner using spanner-cli
Let's quickly connect with CLI.
※We will need to wait a few seconds for the database to be ready.


```
$ docker-compose exec spanner-cli spanner-cli -p test-project -i test-instance -d test-database
Connected.
spanner>
```

We're connected! Now let's check the table.

```bash
spanner> show tables;
+-------------------------+
| Tables_in_test-database |
+-------------------------+
| Singers                 |
| Albums                  |
+-------------------------+
2 rows in set (0.01 sec)

spanner> select * from Singers;
+----------+------------+----------+------------+
| SingerId | FirstName  | LastName | SingerInfo |
+----------+------------+----------+------------+
| 13       | Russell    | Morales  | NULL       |
| 15       | Dylan      | Shaw     | NULL       |
| 12       | Melissa    | Garcia   | NULL       |
| 14       | Jacqueline | Long     | NULL       |
+----------+------------+----------+------------+
4 rows in set (800.4us)
```

We can see tables and data that are created by DB initialization process.

### How to connect from your application

Just set `SPANNER_EMULATOR_HOST=localhost:9010` in the application you are developing. For samples of each client library, please refer to [this official document](https://cloud.google.com/spanner/docs/emulator).


### Emulator Limitations and Differences

As a reminder, the emulator has the following limitations and differences as described in the [official documentation](https://cloud.google.com/spanner/docs/emulator?hl=ja#limitations_and_differences). The following is a list of the points Please understand the following before use.

##### Limitations

> * TLS/HTTPS, authentication, IAM, permissions, roles.
> * PLAN or PROFILE query mode. Only NORMAL is supported.
> * Audit log and monitoring tools.

##### Differences
> * Emulator performance and scalability are not equivalent to production services.
> * Read/write transactions and schema changes lock the entire database for exclusive access only until completion.
> * Partitioned DML and partition queries are supported, but the emulator does not check to see if a statement is partitionable. This means that even if a partitioned DML or partition query statement runs in the emulator, it may fail in production due to a statement error that cannot be partitioned.


## Detailed Explanation

Let me explain the sample code

#### file structure

<img src="/images/20210323/image.png" loading="lazy">

* **docker-compose.yaml** : This is docker-compose file. We will launch container with it.
* **migrations** : Places DDL & DML to be applied during DB initialization

#### Docker images in use

| Docker Image | Description | Usage
 |-----------------------------------------------|---------------------------------------------------------------------------------------------------------|-----------------------------|
| gcr.io/cloud-spanner-emulator/emulator | Spanner emulator provided by GCP [official documentation](https://cloud.google.com/spanner/docs/emulator) | ・Spanner Emulator itself |
| gcr.io/google.com/cloudsdktool/cloud-sdk:slim | Tools and Libraries for Using GCP [Official Documentation](https://cloud.google.com/sdk/docs/downloads-docker) Instance Creation |
| mercari/wrench | Schema management tool for Spanner [Github](https://github.com/cloudspannerecosystem/wrench) | ・Table creation ・Data submission |
| sjdaws/spanner-cli | Spanner's CLI tool [Github](https://github.com/cloudspannerecosystem/spanner-cli) | ・CLI access |

※`wrench` and `spanner-cli` are available at [Cloud Spanner Ecosystem](https://github.com/cloudspannerecosystem)
※ Mercari shares Spanner's tools and findings. Thanks!

#### Diagram of containers  and docker-compose.yaml contents

<img src="/images/20210323/image_2.png" loading="lazy">

spanner emulator itself `spanner` and `spanner-cli` for CLI access will continue to run as resident processes, and other containers will exit normally after command execution

```yaml
version: '3'
services:

    # Spanner
    spanner:
     image: gcr.io/cloud-spanner-emulator/emulator
     ports:
         - "9010:9010"
         - "9020:9020"

    # Init (Create Instance)
    gcloud-spanner-init:
      image: gcr.io/google.com/cloudsdktool/cloud-sdk:slim
      command: >
       bash -c 'gcloud config configurations create emulator &&
               gcloud config set auth/disable_credentials true &&
               gcloud config set project $${PROJECT_ID} &&
               gcloud config set api_endpoint_overrides/spanner $${SPANNER_EMULATOR_URL} &&
               gcloud config set auth/disable_credentials true &&
               gcloud spanner instances create $${INSTANCE_NAME} --config=emulator-config --description=Emulator --nodes=1'
      environment:
        PROJECT_ID: "test-project"
        SPANNER_EMULATOR_URL: "http://spanner:9020/"
        INSTANCE_NAME: "test-instance"
        DATABASE_NAME: "test-database"

    # DB Migration (Create Table)
    wrench-crearte:
      image: mercari/wrench
      command: "create --directory /ddl"
      environment:
        SPANNER_PROJECT_ID: "test-project"
        SPANNER_INSTANCE_ID: "test-instance"
        SPANNER_DATABASE_ID: "test-database"
        SPANNER_EMULATOR_HOST: "spanner:9010"
        SPANNER_EMULATOR_URL: "http://spanner:9020/"
      volumes:
        - ./migrations/ddl:/ddl
      restart: on-failure

    # DB Migration (Insert data)
    wrench-apply:
      image: mercari/wrench
      command: "apply --dml /dml/dml.sql"
      environment:
        SPANNER_PROJECT_ID: "test-project"
        SPANNER_INSTANCE_ID: "test-instance"
        SPANNER_DATABASE_ID: "test-database"
        SPANNER_EMULATOR_HOST: "spanner:9010"
        SPANNER_EMULATOR_URL: "http://spanner:9020/"
      volumes:
        - ./migrations/dml:/dml
      restart: on-failure

    # CLI
    spanner-cli:
      image: sjdaws/spanner-cli:latest
      environment:
        SPANNER_EMULATOR_HOST: "spanner:9010"
      command: ['sh', '-c', 'echo this container keep running && tail -f /dev/null']
```

The following is a supplementary explanation

* wrench container is set to `restart: on-failure
    - We want to run wrench after creating a Spanner instance, but docker-compose's startup control is complicated, so we use fail -> restart -> re-run.
* spanner-cli containers use `tail -f /dev/null` to keep container startup state
    - To execute commands with `docker-exec
    - spanner-cli can also be installed on a local PC by go get (the author had trouble installing it locally).


## Conclusion

As Spanner is called "NewSQL", information on it very little information compared to MySQL and Postgres. But case of its use are increasing in Japan, and I think it is a database service that we can expect to see more of in the future!
