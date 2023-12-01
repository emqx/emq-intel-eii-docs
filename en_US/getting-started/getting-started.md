# Quick Start

## Step 1: Preparation

First, log in to the online file server to download and install the necessary software.

URL: http://47.111.177.175:8080/login

Username: **emqx**

Password: **emqxa303.**

![](../img/2.png)

1. If choosing the "docker image" option for local loading, please download all the files here.

2. If choosing the "docker image" option for online pulling, download only the files outside the `docker-image` directory.

## Step 2: Install Docker

Upload the downloaded software to your Linux server.

1. Install Docker:

```shell
sudo chmod +x get-docker.sh
sudo ./get-docker.sh
```

2. Install Docker Compose:

```shell
sudo install ./docker-compose-linux-x86_64 /usr/local/bin/docker-compose
```

3. Load Images

If you choose to load images locally on the server, load all Docker images separately. You can use the scripts in the `docker-image` directory.

```shell
sudo chmod +x ./load_image.sh
sudo ./load_image.sh
```

## Step 3: Run EII Time Series

1. Unzip `IEdgeInsights.tar.gz`:

```shell
sudo tar -zxvf IEdgeInsights.tar.gz
```

2. Run the `docker-compose` command to start the Docker containers. When all containers are in the "Up" state, it means the startup was successful.

```shell
cd ./IEdgeInsights/build
sudo docker-compose up -d
sudo docker-compose ps -a
```

![Containers](../img/3.png)

3. Run the Modbus simulator:

```shell
mv modbus_linux_amd64 modbus
chmod +x modbus
./modbus
```

## Step 4: Configure Neuron

1. Log in to the Neuron Web console at [http://localhost:7000](http://localhost:7000/) with the username: **admin** and password: **0000.**

2. Add the southbound Modbus TCP plugin.

![Add Plugin](../img/4.png)

   Add the `Modbus TCP` plugin and set its name as `Modbus`.

![Modbus Configuration](../img/5.png)

3. Configure the Modbus driver with the address, port, and other information.

![Modbus Driver Configuration](../img/6.png)

4. Click on **Modbus** driver to add groups and point information. You can import predefined point lists from **Modbus tag.xlsx**.

![Add Groups and Points](../img/7.png)

5. Add the northbound application and select eKuiper.

![Add Northbound Application](../img/8.png)

![Select eKuiper](../img/9.png)

6. Keep the default configuration here. The default port is 7081.

![eKuiper Configuration](../img/10.png)

7. Then, add an eKuiper subscription and choose the southbound Modbus driver plugin you added earlier to report data.

![Add eKuiper Subscription](../img/11.png)

## Step 5: Configure eKuiper

1. Log in to the eKuiper Web console at **http://localhost:9082** with the username: **admin** and password: **public**.

![eKuiper Login](../img/12.png)

3. Add a stream named **NeuronStream** with the stream type as **neuron** and select the default configuration group.

![Add Stream](../img/13.png)

Set the configuration group:

![Stream Configuration](../img/14.png)

4. After configuring, click submit.

![Submit Configuration](../img/15.png)

6. Add a rule with the following settings. If you need to process data, you can refer to the [eKuiper documentation](https://ekuiper.org/docs/zh/latest/).

![Add Rule](../img/16.png)

![Rule Settings](../img/17.png)

The SQL content should be as follows:

```sql
SELECT * from NeuronStream
```

7. Add and configure a sink output for MQTT Sink.

![Add Sink](../img/18.png)

Configure MQTT broker address, MQTT topic, MQTT client ID, MQTT version, QoS, username, password, and other information, then click submit.

![MQTT Sink Configuration](../img/19.png)

## Step 6: Configure EMQX

1. Log in to the EMQX Web console at **http://localhost:18083** with the username: **admin** and password: **public**. You will be prompted to change the password on your first login, so follow the instructions to change it.

2. View MQTT clients.

![MQTT Clients](../img/20.png)

3. Create and configure an InfluxDB resource with the necessary details.

Address: **ia_datastore**

Port: **8086**

Database: **datain**

Username: **admin**

Password: **emqxa303**

![InfluxDB Configuration](../img/21.png)

3. Add a rule engine and configure operations to save data to the InfluxDB database.

![Add Rule Engine](../img/22.png)

Here, use SQL to filter some data points and write them to the InfluxDB database.

```sql
SELECT
  payload.node_name as node_name,
  payload.group_name as group_name,
  payload.timestamp as timestamp,
  payload.values.tag1 as tag1,
  payload.values.tag2 as tag2,
  payload.values.tag3 as tag3
FROM
  "NeuronData"
```

4. Add an action to persist data to InfluxDB. Configure it as follows:

Choose the resource you created earlier and configure the Measurement, timestamp, fields, tags, and other key information.

![InfluxDB Action Configuration](../img/23.png)

![InfluxDB Action Configuration 2](../img/24.png)

## Step 7: Configure Grafana

1. Log in to the Grafana Web console at **https://localhost:3000** with the username: **root** and password: **eii123**.

2. Import the dashboard configuration.

![Import Dashboard](../img/25.png)

Upload and import the local **grafanna.json** file.

![Import Configuration](../img/26.png)

The result will look like this:

![Grafana Dashboard](../img/27.png)

## Reference Documentation

[EMQX Documentation](https://docs.emqx.com/zh/enterprise/v4.4/)

[Neuron Documentation](https://neugates.io/docs/zh/latest/)

[eKuiper Documentation](https://ekuiper.org/docs/zh/latest/)

[EII Documentation](https://eiidocs.intel.com/)

