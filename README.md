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

1.	[IBM Message Hub service](#1-ibm-message-hub-service)
2.	[Produce real-time data into IBM Message Hub](#2-produce-real-time-data-into-ibm-message-hub)
3.	[Consume real-time data from IBM Message Hub](#3-consume-real-time-data-from-ibm-message-hub)
4.  [Explore, transform and analyze real-time data with IBM Data Science Experience](#4-explore-transform-and-analyze-real-time-data-with-ibm-data-science-experience)
5.	[Visualization of real-time system data](#5-visualization-of-real-time-system-data)
6.  [Summary and next steps](#6-summary-and-next-steps)

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

## 4 Explore, transform and analyze real-time data with IBM Data Science Experience
In this section you will learn how to explore, tranform and analyze data with IBM Data Science Experience.

### Data Exploration with PixieDust
In the first step, you can explore your static and real time system data with the Python library PixieDust. This library PixieDust enabling you an deeper insight of the static data. You can inspect the data in a table, create diagrams like bar charts, histograms or scatter plots as well as interactive geo maps with information like the capacity of each station.

#### Explorer - Station information
##### Explore the schema and browse the data
Select DataFrame Table icon in the display widget

##### What are the capacities of the stations?
Choose the Chart icon in the display widget and select and select `Histogram`. Click `Options`
* Values = `capacity`

##### Visualize the capacity of the stations on a map
Choose the Chart icon in the display widget and select and select `Map`. Click `Options`
* Keys = `lat`, `lon`
* Values = `capacity`
* Aggregation = `sum`

Render your map with `mapbox` and insert your own Mapbox access token into your options. You can get a free access token [here](https://www.mapbox.com/studio/account/tokens/)

```
from pixiedust.display import *

#Display station information with PixieDust
display(df_station_info)
```

<p align="left"><img src="https://github.com/danielLinke/CitiBike_NYC/blob/master/images/mapbox.png" width="800"></p>

### Explore - Station status (Animated Table)
You can display your real-time system data from the IBM Message Hub in a animated table. The data will updated continously and will stop after 20 seconds. You can change the duration time of the animation with the parameter `timeout`.

```
import time
from IPython import display
from random import randint

df_station_status = None
timeout = time.time() + 20   # 20 seconds from now
       
#Consume data from IBM Message Hub
for msg in consumer:                 
    if msg.topic == 'citibike':  
        if df_station_status is None:
            df_station_status = pd.DataFrame(_getJSONfromKafka(msg))
            df_station_status = df_station_status.set_index('station_id')  
        else:
            #Update data
            df_temp = pd.DataFrame(_getJSONfromKafka(msg))
            df_temp = df_temp.set_index('station_id')  
            #df_temp['num_bikes_available'] = randint(0, 99) #=>Test Interactive
            df_station_status.update(df_temp, join = 'left', overwrite=True)

        #Display station status
        display.display(df_station_status.head(10))      
        display.clear_output(wait=True)
    
        #Exit condition after 20 seconds
        if time.time() > timeout: 
            break
 ```
 
### Transform Data
In this subsection you will learn how to merge Pandas data frame as preperation for following data analysis. You can find more infromation about structre and realtions of the data [here](https://github.com/NABSA/gbfs/blob/master/gbfs.md).

#### Merge station information with regions in df_station_data
First, you will merge the data frames station information and `system regions` to the data frame `df_station_data`. The data frames will merge via the attribute `region_id`.

```
#Convert region_id in string
if df_station_info['region_id'].dtype == 'float64' or df_station_info['region_id'].dtype == 'int64':
    df_station_info['region_id'] = df_station_info['region_id'].map(lambda x: "{:.0f}".format(x))
    
#Merge station_information with Regions
df_station_data = pd.merge(df_station_info, df_system_regions, how='inner', left_on='region_id', right_index=True)
df_station_data.head(10)
```

#### Merge station data with station status (Animated Table)
First, you will merge the data frames `df_station_data` with the real time system data from IBM Message Hub via the attribute `station_id`.

```
import time
import pylab as plt
from pixiedust.display import *
from IPython import display
from random import randint

df_station_status = None
timeout = time.time() + 20 # 20 seconds from now
    
#Consume data from IBM Message Hub
for msg in consumer:                 
    if msg.topic == 'citibike':  
        if df_station_status is None:
            df_station_status = pd.DataFrame(_getJSONfromKafka(msg))
            df_station_status = df_station_status.set_index('station_id')  
        else:
            #Update data
            df_temp = pd.DataFrame(_getJSONfromKafka(msg))
            df_temp = df_temp.set_index('station_id')  
            #df_temp['num_bikes_available'] = randint(0, 99) #=>Test Interactive
            df_station_status.update(df_temp, join = 'left', overwrite=True)
            
        df_merged = pd.merge(df_station_data, df_station_status, how='inner', left_index=True, right_index=True)
            
        #Display merged table
        display.display(df_merged.head(15))      
        display.clear_output(wait=True)
                   
        #Exit condition after 20 seconds
        if time.time() > timeout: 
            break
```

#### Analyze Data
You will get insights into the provided bike sharing data of Citi Bike. You will analysis the capacity, availability and utilization of the bike sharing stations in New York City and New Jersey.

##### Analyze data in PixieDust

##### Explore the schema and browse the data
Select DataFrame Table icon in the display widget

##### What are the current bike availability of the stations?
Choose the Chart icon in the display widget and select `Histogram`. Click `Options`
* Values = `num_bikes_available`

##### How many bikes currently are available and rented in New York City and Jersey City
Choose the Chart icon in the display widget and select `Bar chart`. Click `Options`
* Keys = `name_y`
* Values = `num_bikes_available`, `cacpacity`

##### Visualize the bike availability of the stations on a map
Choose the Chart icon in the display widget and select `Map`. Click `Options`
* Keys = `lat`, `lon`
* Values = `num_bikes_available`
* Aggregation = `sum`

##### Visualize the disabled bikes and docks on a map
Choose the Chart icon in the display widget and select `Map`. Click `Options`
* Keys = `lat`, `lon`
* Values = `num_bikes_disabled`, `num_docks_disabled`
* Aggregation = `sum`

Render your map with `mapbox` and insert your own Mapbox access token into your options. You can get a free access token [here](https://www.mapbox.com/studio/account/tokens/)

```
from pixiedust.display import *

#Display merged data with PixieDust
display(df_merged)
```
<p align="left"><img src="https://github.com/danielLinke/CitiBike_NYC/blob/master/images/mapbox2.png" width="800"></p>

### Histogram - Available Bikes (Animated Plot)
In the following animated histogram, you can see the availibilty distribution of all bike sharing stations in real time. The diagram using real time system data from the IBM Message Hub service and will stop after 20 seconds. You can change the duration time of the animation with the parameter `timeout`.

<p align="left"><img src="https://github.com/danielLinke/CitiBike_NYC/blob/master/images/histogram.png" width="500"></p>

### Data Plots - Utilisation and Relations
With the following figures, you can get an deeper insight of the bike sharing data. The figures showing an overview about the utilization for each district as well further statistical information about the stations in boxplots and different kind of scatterplots.

```
%matplotlib inline
import matplotlib.pyplot as plt

#Set Display size for output
plt.rcParams["figure.figsize"] = [22, 20]

#Set Subplot settings (3x3)
fig, axes = plt.subplots(nrows=3, ncols=3)

#Consume data from IBM Message Hub
for msg in consumer:                 
    if msg.topic == 'citibike':  
        df_station_status = pd.DataFrame(_getJSONfromKafka(msg))
        df_station_status = df_station_status.set_index('station_id')       
        
        #Merge data
        df = pd.merge(df_station_data, df_station_status, how='inner', left_index=True, right_index=True)
        df['utilization'] = (df.num_bikes_available / df.capacity)*100 
        
        #DataFrames grouped by district
        df_overall = df.groupby(['region_id'])[['num_docks_available', 'num_bikes_available', 'num_bikes_disabled', 'num_docks_disabled', 'utilization']].mean()           
            
        #Boxplots
        ax1 = df[['num_bikes_available', 'num_docks_available', 'capacity', 'utilization']].plot.box(vert=False, sym='r+', ax=axes[0,0])
        ax1.set_title('Utilization', fontweight='bold')
        ax1.set_xlabel('Percent [%]')
        
        #BarPlot - Overall utilization  
        ax2 = df_overall[['num_docks_available', 'num_bikes_available', 'num_bikes_disabled', 'num_docks_disabled']].plot.bar(legend=True, stacked=False, ax=axes[0,1]) 
        ax2.set_title('Overall utilization', fontweight='bold')
        ax2.set_xticklabels(['Jersey', 'NYC'], rotation=0)
        ax2.set_xlabel('Regions')
        ax2.set_ylabel('Amount')
    
        #BarPlot - Realtive utilization
        ax3 = df_overall['utilization'].plot.bar(legend=True, ax=axes[0,2]) 
        ax3.set_title('Relative utilization', fontweight='bold') 
        ax3.set_xticklabels(['Jersey', 'NYC'], rotation=0)
        ax3.set_xlabel('Regions')
        ax3.set_ylabel('Percent [%]')
            
        #Pie Chart - Number of Stations       
        values = [3, 12] 
        ax9 = df.region_id.value_counts().plot(kind='pie', autopct='%.2f', ax=axes[1,0], labels=['NYC', 'Jersey'], shadow=True)
        ax9.set_title('Number of stations in each region ', fontweight='bold')
        ax9.set_xlabel('Percent [%]')
        ax9.set_ylabel('Regions')     
               
        #Scatter Plot - Latitiude
        ax5 = df.plot.scatter('lat', 'utilization', c='blue', ax=axes[1,1])
        ax5.set_title('Relation utilization and latidude', fontweight='bold')
        
        #Scatter Plot - Longitude
        ax6 = df.plot.scatter('lon', 'utilization', c='blue', ax=axes[1,2])
        ax6.set_title('Relation utilization and longitude', fontweight='bold')
        
        #Scatter Plot - Capacity
        ax4 = df.plot.scatter('capacity', 'utilization', c='green', ax=axes[2,0])
        ax4.set_title('Relation capacity and utilization', fontweight='bold') 
        
        #Scatter Plot - Key dispender
        ax7 = df.plot.scatter('utilization', 'eightd_has_key_dispenser', c='black', ax=axes[2,1])
        ax7.set_title('Relation utilization and eightd_has_key_dispenser', fontweight='bold')
                
        #Scatter Plot - Docks and Bikes
        ax8 = df.plot.scatter('num_docks_available', 'num_bikes_available', c='red', ax=axes[2,2])       
        ax8.set_title('Relation num_docks_available and num_bikes_available', fontweight='bold')
        
        break
```

<p align="left"><img src="https://github.com/danielLinke/CitiBike_NYC/blob/master/images/dataplots.png" width="800"></p>

#### Correlations
In the following table, you can see the correlation beetween all attributes of the bike sharing data. A correlation coefficient of 1 or -1 mean a high correlation beetween the attributes. A correlation coefficient around 0 stand for a low correlation.

```
%matplotlib inline
import matplotlib.pyplot as plt
import seaborn as sns
            
#Consume data from IBM Message Hub
for msg in consumer:                 
    if msg.topic == 'citibike':  
        df_station_status = pd.DataFrame(_getJSONfromKafka(msg))
        df_station_status = df_station_status.set_index('station_id')       
        
        #Merge data
        df = pd.merge(df_station_data, df_station_status, how='inner', left_index=True, right_index=True)
        df['utilization'] = (df.num_bikes_available / df.capacity)*100 
        df_corr = df.corr()                
        
        #Display table
        df_corr = df_corr.style.background_gradient(cmap=plt.get_cmap('coolwarm'), axis=1)       
        break
df_corr
```

### 5 Visualization of real-time system data
In this section, you will learn how to visualize heat maps of real-time system data. You will using the python library `mplleaflet` to create geo maps with [OpenStreetMap](https://www.openstreetmap.org/about). Through the maps, you get a better insight of the capacity, availibilty and utilization of the bike station depending on the station location.

#### Load GeoJson Districts
Download the different districts of New York and Jersey as geojson maps. You will embedded the geojson files in the following mplleaflet maps.

```
import geojson
from descartes import PolygonPatch
import matplotlib.pyplot as plt
import numpy as np

#GeoMaps for district in NYC and Jersey
!wget -q 'https://data.cityofnewyork.us/api/geospatial/tqmj-j8zm?method=export&format=GeoJSON' -O district_nyc.geojson
!wget -q 'http://catalog.civicdashboards.com/dataset/9c9c69f3-42f3-4f1e-bdb3-d68bba2ebfd2/resource/657066c7-051e-4606-802f-0ca92d5a18bc/download/aeda17dbe956493a826a2c8b025d67a0njcountypolygonv2.geojson' -O district_jersey.geojson

#Read GeoJson files
with open('district_nyc.geojson') as json_file:
    geojson_nyc = geojson.load(json_file)
    
with open('district_jersey.geojson') as json_file:
    geojson_jersey = geojson.load(json_file)
    
#Define district information and colors
district_nyc = geojson_nyc['features']
district_jersey = geojson_jersey['features']
BLUE = '#6699cc'
GREEN = '#7FFF7F'
RED = '#EC3B00'
YELLOW = '#FFF400'
BLACK = '#000000'
CYAN = '#00FFDF'
ORANGE = '#FFA500'
```

#### GeoMap - Capacity
The following map shows the capacity for each bike station. Station with a high capacity over 40 are green. Station with a capacity between 20 and 40 are yellow. All stations with a low capcity under 20 bikes are red. In the map you can recoginze, bike stations with a low capacity are mostly in Jersey City or on the outskirts of New York City. On the contrary the stations with a high capcity are predominated in the district Manhatten.

```
%matplotlib inline
import mplleaflet
import matplotlib.pyplot as plt
from descartes import PolygonPatch

#Set display size for output
plt.rcParams["figure.figsize"] = [15, 8]
     
#Define stations according to the capacity
df_low = df_station_info[df_station_info['capacity'] <= 20] #Stations with low capacity
df_middle = df_station_info[df_station_info['capacity'].between(20, 40, inclusive=False)] #Stations with middle capacity 
df_high = df_station_info[df_station_info['capacity'] >= 40] #Stations with high capacity          

#Plot capacity stations
f, ax = plt.subplots()                            
plt.plot(df_low['lon'], df_low['lat'], 'rs', label='Line 3')  # Red squares for stations with low capacity
plt.plot(df_middle['lon'], df_middle['lat'], 'ys', label='Line 2') # Yellow squares for stations with middle capacity
plt.plot(df_high['lon'], df_high['lat'], 'gs', label='Line 1') # Green squares for stations with high capacity

#Plot districts
queens = ax.add_patch(PolygonPatch(district_nyc[0]['geometry'], fc=CYAN, ec=BLACK, alpha=0.5, zorder=2))
#statenIsland = ax.add_patch(PolygonPatch(district_nyc[1]['geometry'], fc=ORANGE, ec=BLACK, alpha=1, zorder=2))
#bronx = ax.add_patch(PolygonPatch(district_nyc[2]['geometry'], fc=BLUE, ec=BLACK, alpha=1, zorder=2))
brooklyn = ax.add_patch(PolygonPatch(district_nyc[3]['geometry'], fc=YELLOW, ec=BLACK, alpha=1, zorder=2))
manhatten = ax.add_patch(PolygonPatch(district_nyc[4]['geometry'], fc=RED, ec=BLACK, alpha=1, zorder=2))
jersey = ax.add_patch(PolygonPatch(district_jersey[1]['geometry'], fc=GREEN, ec=BLACK, alpha=1, zorder=2))

#Show map
mplleaflet.display(fig=f)
```

<p align="left"><img src="https://github.com/danielLinke/CitiBike_NYC/blob/master/images/geomap.png" width="800"></p>

#### Heat Map - Available Bikes
The following heat map shows the availibilty of bikes for each bike station.

```
%matplotlib inline
import mplleaflet
import matplotlib.pyplot as plt
import matplotlib.cm as cm

#Set display size for output
plt.rcParams["figure.figsize"] = [15, 8]

#Consume data from IBM Message Hub
for msg in consumer:              
    if msg.topic == 'citibike':      
        df_station_status = pd.DataFrame(_getJSONfromKafka(msg))
        df_station_status = df_station_status.set_index('station_id')
        
        #Merge data
        df = pd.merge(df_station_data, df_station_status, how='inner', left_index=True, right_index=True)
                                               
        #Plot availibilty stations                                    
        f, ax = plt.subplots()
        plt.scatter(x=df['lon'], y=df['lat'], c=df['num_bikes_available'], s=100+(100/(df['num_bikes_available']+1)), cmap=cm.get_cmap('hsv'), edgecolor='None')           
        
        #Plot districts
        queens = ax.add_patch(PolygonPatch(district_nyc[0]['geometry'], fc=CYAN, ec=BLACK, alpha=0.5, zorder=2))
        #statenIsland = ax.add_patch(PolygonPatch(district_nyc[1]['geometry'], fc=ORANGE, ec=BLACK, alpha=1, zorder=2))
        #bronx = ax.add_patch(PolygonPatch(district_nyc[2]['geometry'], fc=BLUE, ec=BLACK, alpha=1, zorder=2))
        brooklyn = ax.add_patch(PolygonPatch(district_nyc[3]['geometry'], fc=YELLOW, ec=BLACK, alpha=1, zorder=2))
        manhatten = ax.add_patch(PolygonPatch(district_nyc[4]['geometry'], fc=RED, ec=BLACK, alpha=1, zorder=2))
        jersey = ax.add_patch(PolygonPatch(district_jersey[1]['geometry'], fc=GREEN, ec=BLACK, alpha=1, zorder=2))
               
        break;
mplleaflet.display(fig=f)
```
<p align="left"><img src="https://github.com/danielLinke/CitiBike_NYC/blob/master/images/heatmap_available.png" width="800"></p>

#### HeatMap - Utilization
The following heat map shows the utilization for each bike station.

```
%matplotlib inline
import mplleaflet
import matplotlib.pyplot as plt
import matplotlib.cm as cm

#Set display size for output
plt.rcParams["figure.figsize"] = [15, 8]

#Consume data from IBM Message Hub
for msg in consumer:              
    if msg.topic == 'citibike':      
        df_station_status = pd.DataFrame(_getJSONfromKafka(msg))
        df_station_status = df_station_status.set_index('station_id')
        
        #Merge data
        df = pd.merge(df_station_data, df_station_status, how='inner', left_index=True, right_index=True)
        df['utilization'] = (df.num_bikes_available / df.capacity)*100      
        
        #Plot data                                    
        f, ax = plt.subplots()
        plt.scatter(x=df['lon'], y=df['lat'], c=df['utilization'], s=df['utilization']*5, cmap=cm.get_cmap('coolwarm'), edgecolor='None')         
        
        #Plot districts
        queens = ax.add_patch(PolygonPatch(district_nyc[0]['geometry'], fc=CYAN, ec=BLACK, alpha=0.5, zorder=2))
        #statenIsland = ax.add_patch(PolygonPatch(district_nyc[1]['geometry'], fc=ORANGE, ec=BLACK, alpha=1, zorder=2))
        #bronx = ax.add_patch(PolygonPatch(district_nyc[2]['geometry'], fc=BLUE, ec=BLACK, alpha=1, zorder=2))
        brooklyn = ax.add_patch(PolygonPatch(district_nyc[3]['geometry'], fc=YELLOW, ec=BLACK, alpha=1, zorder=2))
        manhatten = ax.add_patch(PolygonPatch(district_nyc[4]['geometry'], fc=RED, ec=BLACK, alpha=1, zorder=2))
        jersey = ax.add_patch(PolygonPatch(district_jersey[1]['geometry'], fc=GREEN, ec=BLACK, alpha=1, zorder=2))
        
        break;
mplleaflet.display(fig=f) 
```
<p align="left"><img src="https://github.com/danielLinke/CitiBike_NYC/blob/master/images/heatmap2.png" width="800"></p>

### 6 Summary and next steps
You successfully completed this notebook! You learned how to use IBM Message Hub, Pandas, PixieDust as well as Mplleaflet for consuming, analysing and visualizing real-time system data.

#### Next Steps
Using historical bike sharing data and weather data for deeper insights.
