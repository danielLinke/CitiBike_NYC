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
3.	[Consume real-time data from IBM Message Hub](#consumer-)
4.  [Explore, transform and analyze real-time data with IBM Data Science Experience](#analyze-)
5.	[Visualization of real-time system data](#visualization-)
6.  [Summary and next steps](#summary-)

## 1 IBM Message Hub service <a name="message-hub"></a>
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


`
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
`

## 2 Produce real-time data into IBM Message Hub <a name="producer"></a>
In the next step, we will use an application to produce a real-time data stream into the IBM Message Hub service. The producer application is written in Java. In the scenario, the application read real-time system data from the [Citi bike feed](https://gbfs.citibikenyc.com/gbfs/gbfs.json) and send this datastream to the topic `citibike` into IBM's Message Hub service. For a deeper insight of the producer application. You can find source code and more in the Message Hub samples [here](https://github.com/danielLinke/message-hub-samples).

### Running the producer sample
