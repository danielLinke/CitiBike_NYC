# Analyze real-time system data from Citi Bike with IBM Message Hub

[Citi Bike](https://www.citibikenyc.com/) is the largest bike share program in New York City with around 10,000 bikes and 600 stations across Manhatten, Brookyn, Queens and Jersey City. The platform publishes real-time system data in [General Bikeshare Feed Specification (GBFS)](https://github.com/NABSA/gbfs/blob/master/gbfs.md) format. [Get the GBFS feed here.](https://gbfs.citibikenyc.com/gbfs/gbfs.json)  

In this tutorial, we will use [IBM Data Science Experience](https://datascience.ibm.com/) to analyze and extract insights from this real-time system data of Citi Bike. For the prupose, we use technologies like [Jupyter Notebooks](http://jupyter.org/) and [IBM Message Hub](http://www-03.ibm.com/software/products/en/ibm-message-hub). The objective of the tutorial is to demonstrate, the process of producing and consuming real-time data with IBM Message Hub as well as the analysis and visualization of real-time system data with IBM Data Science Experience.

When the reader has completed this tutorial, they will understand how to:

* Create and configure the IBM Message Hub services on Bluemix.
* Produce real-time data into IBM Message Hub service.
* Consume real-time data from IBM Message Hub service.
* Transform and analyze real-time data with Jupyter notebooks in IBM Data Science Experience.
* Visualization of real-time system data.

## Included Components
* IBM Data Science Experience
* Bluemix for IBM Message Hub
* Pandas (Python data analysis library)
* Matplotlib (Python 2D plotting library)
* PixieDust (Python helper library)
* Mplleaflet (Python library for geo visualization)

# Contents
This notebook contains the following parts:

1.	[IBM Message Hub service](#1-ibm-message-hub-service-)
2.	[Produce real-time data into IBM Message Hub](#2-produce-real-time-data-into-ibm-message-hub-)
3.	[Consume real-time data from IBM Message Hub](#3-consume-real-time-data-from-ibm-message-hub)
4.  [Explore, transform and analyze real-time data with IBM Data Science Experience](#analyze-)
5.	[Visualization of real-time system data](#visualization-)
6.  [Summary and next steps](#summary-)

## 1 IBM Message Hub service
IBM Message Hub is a cloud service from Bluemix and based on Apache Kafka: a fast, scalable, and durable real-time messaging engine from the Apache Software Foundation. To understand the benefits of using Apache Kafka as a service, see [Message Hub: Apache Kafka as a Service](https://developer.ibm.com/messaging/2016/03/14/message-hub-apache-kafka-as-a-service/).

### Sign up for the Data Science Experience
If you don't have an account for Data Science Experience, yet. Please, sign up for [IBM's Data Science Experience](https://datascience.ibm.com/). The IBM Data Science Experience is free for a 30-day trial period.

### Create an IBM Message Hub service
Now you’re ready to start laying down infrastucture for live-streaming. Start by creating a Message Hub instance. You can create your IBM Message Hub instance here or following the instruction below.

1. Go into your IBM Data Science Experience account
2. In Data Science Experience, on top menu, click `Data Services` and afterwards `Services`.
3. You can create an IBM Message Hub instance under the button `Create new` and select `Message Hub`.
4. Click `Buy Message Hub` for creating a new Message Hub instance
5. Use the default settings, insert your service name (for example: `message-hub`) and confirm your settings

Now, you have created an IBM Message Hub instance successfully in Bluemix. You can also using an exisiting Message Hub instance from Bluemix or create a Message Hub instance directly in Bluemix.

### Get your Service credentials from IBM Message Hub service

In this step, you will insert your service credentials from the IBM Message Hub instance into your notebook.

1. Go into your IBM data Science Experience Notebook
2. → Insert a cell below with this button and go into the cell
3. → Click this Button on top menu
4. In the side bar on the right side, Click `Connections`
5. You will see the service_name of your Message Hub instances. Click for your Service Hub instances `insert to code`.
6. The new generated code should look like in the example below and contains the parameters `api_key`,`kafka_admin_url`, `kafka_broker_sasl`, `user` and `password`. The URLs may differ depending on your Bluemix region.


```
credentials_1 = {
  'instance_id\':'fa0e5d69-995a-4281-ab1e-48fd017742c8',
  'mqlight_lookup_url':'https://mqlight-lookup-prod01.messagehub.services.us-south.bluemix.net/Lookup?serviceId=fa0e5d69-995a-4281-ab1e-48fd017742c8',
  'api_key':'<your_api_key>',
  'kafka_admin_url':'https://kafka-admin-prod01.messagehub.services.us-south.bluemix.net:443',
  'kafka_rest_url':'https://kafka-rest-prod01.messagehub.services.us-south.bluemix.net:443',
  'kafka_brokers_sasl':'kafka01-prod01.messagehub.services.us-south.bluemix.net:9093,kafka04-prod01.messagehub.services.us-south.bluemix.net:9093,kafka03-prod01.messagehub.services.us-south.bluemix.net:9093,kafka05-prod01.messagehub.services.us-south.bluemix.net:9093,kafka02-prod01.messagehub.services.us-south.bluemix.net:9093',
  'user':'<your_user>',
  'password':'<your_password>',
  'topic':'citibike'
}
```

## 2 Produce real-time data into IBM Message Hub
In the next step, we will use an application to produce a real-time data stream into the IBM Message Hub service. The producer application is written in Java. In the scenario, the application read real-time system data from the [Citi bike feed](https://gbfs.citibikenyc.com/gbfs/gbfs.json) and send this datastream to the topic `citibike` into IBM's Message Hub service. For a deeper insight of the producer application. You can find source code and more in the Message Hub samples [here](https://github.com/danielLinke/message-hub-samples).

### Running the producer sample
Please, download the zip file [kafka producer](https://github.com/danielLinke/CitiBike_NYC/blob/master/kafka-producer.zip). Save and unzip the file on your local device. Afterwards, go and change to the directory of the file `kafka-producer.jar` and execute the following command in your console:

```
java -cp kafka-producer.jar <kafka_brokers_sasl> <kafka_admin_url> <api_key> -topic "citibike" -producer -json "https://gbfs.citibikenyc.com/gbfs/en/station_information.json"
```

To find the values for `<kafka_brokers_sasl>`, `<kafka_admin_url>` and `<api_key>`, access your Message Hub instance in Bluemix, go to the `Service Credentials` tab and use your indivdiual `Credentials`.

__Note:__ `<kafka_brokers_sasl>` must be a single string enclosed in quotes. For example: `"host1:port1,host2:port2"`. We recommend using all the Kafka hosts listed in the `Credentials` you selected.

You run only the producer by respectively adding -producer to the command above. Also, we specify the Kafka topic to use by adding the argument `-topic "citibike"` and the JSON source file with `-json "https://gbfs.citibikenyc.com/gbfs/en/station_information.json"`.

The sample will run indefinitely until interrupted. To stop the process, use `Ctrl + C`, for example. Below is a snippet of the output generated by the sample, if the applciation works fine.

```
...
[2017-07-05 15:36:01,496] INFO Creating the topic citibike (com.messagehub.samples.MessageHubConsoleSample)
[2017-07-05 15:36:04,044] INFO Admin REST Listing Topics: [{"name":"citibike","partitions":1,"retentionMs":"86400000","markedForDeletion":false}] (com.messagehub.samples.MessageHubConsoleSample)
[2017-07-05 15:36:05,314] INFO [Partition(topic = citibike, partition = 0, leader = 6, replicas = [5,6,7], isr = [6,5,7])] (com.messagehub.samples.ProducerRunnable)
[2017-07-05 15:36:05,315] INFO MessageHubConsoleSample will run until interrupted. (com.messagehub.samples.MessageHubConsoleSample)
[2017-07-05 15:36:05,317] INFO class com.messagehub.samples.ProducerRunnable is starting. (com.messagehub.samples.ProducerRunnable)
[2017-07-05 15:36:09,034] INFO Message produced, offset: 16740 (com.messagehub.samples.ProducerRunnable)
[2017-07-05 15:36:11,934] INFO Message produced, offset: 16741 (com.messagehub.samples.ProducerRunnable)
[2017-07-05 15:36:15,176] INFO Message produced, offset: 16742 (com.messagehub.samples.ProducerRunnable)
[2017-07-05 15:36:17,586] INFO Message produced, offset: 16743 (com.messagehub.samples.ProducerRunnable)
[2017-07-05 15:36:19,964] INFO Message produced, offset: 16744 (com.messagehub.samples.ProducerRunnable)
[2017-07-05 15:36:22,955] INFO Message produced, offset: 16745 (com.messagehub.samples.ProducerRunnable)
[2017-07-05 15:36:25,790] INFO Message produced, offset: 16746 (com.messagehub.samples.ProducerRunnable)
[2017-07-05 15:36:28,167] INFO Message produced, offset: 16747 (com.messagehub.samples.ProducerRunnable)
[2017-07-05 15:36:30,567] INFO Message produced, offset: 16748 (com.messagehub.samples.ProducerRunnable)
...
```

## 3 Consume real-time data from IBM Message Hub
In this section you will load the data as an PandasData Frame. The static data from Citi Bike like station information and system regions will loaded by using the HTTP library `requests` for Python. For consuming the real-time system data from the IBM Message Hub service, you will use the `KafkaConsumer`, a Kafka client that consumes records from a Kafka cluster like IBM Message Hub.

### Preparation - Install Libraries
Before you use the sample code in this notebook, you must install the libararies `kafka-python`, `pixiedust` and `mplleaflet` to your notebook. You can install the libraries via the package management system `pip`.

```
#Install libraries
!pip install --user kafka-python
!pip install --user mplleaflet
!pip install --user geojson
!pip install --user IPython
!pip install --user --upgrade pixiedust
```
### Load static data from CitiBike
Afterwards you can download the static data as an Pandas DataFrame directly from Citi Bike. You can find the data station information and system regions under the following URLs: [https://gbfs.citibikenyc.com/gbfs/en/station_information.json](https://gbfs.citibikenyc.com/gbfs/en/station_information.json) and [https://gbfs.citibikenyc.com/gbfs/en/system_regions.json](https://gbfs.citibikenyc.com/gbfs/en/system_regions.json).

```
import requests
import json
import pandas as pd
from pandas.io.json import json_normalize

#Get JSON with Requests
def _getDataFramefromURL(url, entity, index):
    response = requests.request("GET", url)
    json_response = json.loads(response.text)
    df_temp = pd.DataFrame(json_normalize(json_response['data'][entity]))
    df_temp = df_temp.set_index(index) 
    return df_temp

# Get static data from CitiBike from URL
url_station_info = "https://gbfs.citibikenyc.com/gbfs/en/station_information.json"
url_system_regions = "https://gbfs.citibikenyc.com/gbfs/en/system_regions.json"

df_station_info = _getDataFramefromURL(url_station_info, 'stations', 'station_id')
df_system_regions = _getDataFramefromURL(url_system_regions, 'regions', 'region_id')
```

### Load real-time system data from IBM MessageHub
Additionally, you consume the real-time system data from the IBM Message Hub with a Kafka consumer. The Kafka consumer should read the topic 'citibike' from your Message Hub service. Notice, the values of the parameters `bootstrap_servers` , `sasl_plain_username` and `sasl_plain_password` are from your own service credentials of the IBM Message Hub service.

```
from kafka import KafkaConsumer
from kafka import TopicPartition
import ssl
import json
from pprint import pprint

import numpy as np
import pandas as pd
from pandas.io.json import json_normalize

from IPython.display import display, HTML

############################################
# Service credentials from Bluemix UI:
############################################
bootstrap_servers =   credentials_1.get('kafka_brokers_sasl')
sasl_plain_username = credentials_1.get('user')
sasl_plain_password = credentials_1.get('password')
############################################

sasl_mechanism = 'PLAIN'
security_protocol = 'SASL_SSL'

def _getJSONfromKafka(msg):
    #Parse JSON from Kafka Stream    
    station_info = json.loads(msg.value.decode("utf-8"))
    return json_normalize(station_info['data']['stations']) 

# Create a new context using system defaults, disable all but TLS1.2
context = ssl.create_default_context()
context.options &= ssl.OP_NO_TLSv1
context.options &= ssl.OP_NO_TLSv1_1

consumer = KafkaConsumer(bootstrap_servers = bootstrap_servers,
                         sasl_plain_username = sasl_plain_username,
                         sasl_plain_password = sasl_plain_password,
                         security_protocol = security_protocol,
                         ssl_context = context,
                         sasl_mechanism = sasl_mechanism,
                         api_version=(0,10))

#Set topic
consumer.assign([TopicPartition('citibike', 0)])
```
