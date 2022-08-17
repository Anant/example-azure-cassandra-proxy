# Azure Dual Write Cassandra Proxy
Learn how to use the Azure Dual Write Cassandra Proxy. This demo is designed to be done on Gitpod (on your browser and for free!). You can however do this demo on your local using docker, but you may need to adjust some commands that return IP addresses. 

## Click below to get started!

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/Anant/example-azure-cassandra-proxy)

### 1. Confirm the installs completed and Both Dockerized Cassandra has started
Each task in the `.gitpod.yml` file run in their own terminals that will start on Gitpod load. 

#### 1.1 Start CQLSH in source and dest Cassandra containers
```bash
docker exec -it source cqlsh
```

```bash
docker exec -it dest cqlsh
```

### 2. Run `data_importer.py` for dockerized Cassandra Source Container
#### 2.1 Run the below command with your desired keyspace and table names after replacing the placeholders
```bash
python data_importer.py $(hostname -I | awk '{print $2}') <keyspace_name> <table_name>
```

#### 2.2 Confirm your keyspace and table were created and data was loaded in Source Container CQLSH
```bash
select count(*) from <keyspace_name>.<table_name>
```

This should return 10

#### 2.3 Drop the keyspace you just created
```bash
drop keyspace <keyspace_name> ;
```

#### 2.4 Confirm dest Cassandra container is still unchanged via CQLSH
```bash
describe keyspaces ;
```

Confirm that they <keyspace_name> was not created in the dest Cassandra container. 

### 3. Start Cassandra Proxy
#### 3.1 Run the below command to start the Cassandra Proxy

```bash
java -jar cassandra-proxy-1.0-SNAPSHOT-fat.jar --source-port 9042 --target-port 9043 $(hostname -I | awk '{print $2}') $(hostname -I | awk '{print $2}')
```

#### 3.2 Update Port to Point to Cassandra Proxy in Data Importer
```bash
sed -i 's/port=9042/port=29042/' data_importer.py
```

### 4. Re-run `data_importer.py` using the dual write Cassandra Proxy

#### 4.1 Run the below command with your keyspace and table name
```bash
python data_importer.py $(hostname -I | awk '{print $2}') <keyspace_name> <table_name>
```

#### 4.2 Wait, why didn't that work?
You may see an error such as the one below:
```console

```
This is most likely due to TLS related issues; however, we can disable that using flags in the command.

#### 4.3 Kill and restart Cassandra Proxy
```bash
ctrl + c in Cassandra Proxy terminal
```

```bash
java -jar cassandra-proxy-1.0-SNAPSHOT-fat.jar --source-port 9042 --target-port 9043 --disable-source-tls true --disable-target-tls true $(hostname -I | awk '{print $2}') $(hostname -I | awk '{print $2}')
```

then

```bash
python data_importer.py $(hostname -I | awk '{print $2}') <keyspace_name> <table_name>
```

### 5. Confirm Cassandra Proxy dual wrote to both source and dest containers

#### 5.1 Run a count command source CQLSH terminal
```bash
select count(*) from <keyspace_name>.<table_name>
```

#### 5.2 Run a count command dest CQLSH terminal
```bash
select count(*) from <keyspace_name>.<table_name>
```

Both commands should return 10. And that's how we can use the Azure Cassandra Proxy on two dockerized instances of Apache Cassandra.
