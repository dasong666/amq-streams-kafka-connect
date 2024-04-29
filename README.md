# amq-streams-kafka-connect

![Architecture](./kafka-connect-overview.png)

## Prerequisites:

Install the following operators
* AMQ Streams

Kafka Broker/Bootstrap/Zookeeper Cluster
* Assumes you already have AMQ Streams Strimzi Kafka Cluster running successfully

Source and Sink Databases
* Instruction steps below include how to provision these databases: MSSQL, MongoDB

To use the yaml examples in this lab:
* Replace <your-project> with your own project namespace
* Make sure kafka resources and cluster names match your project

External image registry and source repository access
* Make sure your environment allows your OpenShift cluster to reach out to external image registries and Maven repositories


## Apply the CRD resources in this order
```
KafkaConnect:
oc -n <your-namespace> apply -f yaml/kafka-connect/kafka-connect-metrics-example.yaml
oc -n <your-namespace> apply -f yaml/kafka-connect/kafka-connect.yaml
oc -n <your-namespace> apply -f yaml/kafka-connect/kafka-connector-sqlserver-debezium.yaml 
oc -n <your-namespace> apply -f yaml/kafka-connect/kafka-connector-mongodb.yaml

helm repo add bitnami https://charts.bitnami.com/bitnami

helm install mongodb bitnami/mongodb \
--set architecture=replicaset \
--set replicaCount=3 \
--set podSecurityContext.fsGroup="",containerSecurityContext.enabled=false,podSecurityContext.enabled=false,auth.enabled=false \
--version 13.6.0 \
-n <your-namespace>

MSSQL:
oc new-project mssql
oc -n mssql apply -f yaml/mssql/mssql-data.yaml
oc -n mssql apply -f yaml/mssql/mssql-sql.yaml

The following step must be completed as cluster-admin:

oc -n mssql adm policy add-scc-to-user anyuid -z default

oc -n mssql apply -f yaml/mssql/mssql-deployment.yaml
oc -n mssql apply -f yaml/mssql/mssql-service.yaml

oc -n mssql exec $(oc -n mssql get pods -l 'deployment=dbserver' -o jsonpath='{.items[0].metadata.name}') -- /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P 'Abcd1234' -i /opt/workshop/mssql-sql.sql

```

## Post Installation Steps
Insert some item descriptions into the MS SQL Server DB.

```
oc -n mssql run sql-server -ti --image=mcr.microsoft.com/mssql/server:2022-latest --rm=true --restart=Never -- /opt/mssql-tools/bin/sqlcmd -S server.mssql.svc -U sa -P 'Abcd1234'
```

```
use OrdersDB
go

insert into ItemDescription values ('In-Progress', 'Cogs')
insert into ItemDescription values ('In-Progress', 'Sprockets')
insert into ItemDescription values ('In-Progress', 'Ball Bearings')
insert into ItemDescription values ('In-Progress', 'Rotator Splints')
insert into ItemDescription values ('In-Progress', 'Doodads')
go

```
