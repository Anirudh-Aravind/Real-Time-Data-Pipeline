# Real-Time-Data-Pipeline
The aim of this project is to create a real-time data pipeline for
processing e-commerce data using Apache Kafka and Apache
Cassandra. Here I am ingesting data from a CSV file using a Kafka producer,
transform the data using a Kafka consumer, and finally store the
processed data in a Cassandra table.

Steps:
1. Dataset:
   
   Load the 'olist_orders_dataset.csv' into a pandas dataframe and
   examine its structure and contents.
2. Apache Kafka Setup:
   
   Install and configure Apache Kafka on your system. Create a
   Kafka topic, named 'ecommerce-orders', to hold the e-commerce
   data
   
   * To use Kafak, here I used Confluent Kafka. Create an account in here
     
     URL: https://www.confluent.io/get-started/?_gl=1*jdahll*_ga*MTkwODQzNDkzNS4xNjk2Njg0Nzgy*_ga_D2D3EGKSGD*MTY5Njc0ODA1Ni4zLjAuMTY5Njc0ODA1Ny41OS4wLjA.&_ga=2.184538854.1910110190.1696684782-1908434935.1696684782&_gac=1.180626901.1696692366.CjwKCAjwg4SpBhAKEiwAdyLwvCXfyGwmmAl7f-        tGKjjwEBfwKTzwDjJqwZMKqENnGSeoTXq0Y5NprRoC-PUQAvD_BwE

     Select the Environments option and click on the Add cloud environment to create a new environment

     ![Screenshot 2023-10-08 123142](https://github.com/Anirudh-Aravind/Real-Time-Data-Pipeline/assets/84184475/9cab5ddd-f433-479f-bdd5-149e2f932e17)


     Create a cluster under the above created environment

     ![Screenshot 2023-10-08 123840](https://github.com/Anirudh-Aravind/Real-Time-Data-Pipeline/assets/84184475/51e164fa-e64a-4e20-9395-804bf894cdc9)

     Click on the API Keys option under the above created Cluster and and save that file in local. This file is required to access the Kafka programmatically
     ![Screenshot 2023-10-08 124245](https://github.com/Anirudh-Aravind/Real-Time-Data-Pipeline/assets/84184475/cc0c0675-69b1-4fb4-a51d-05fe105f746e)


    Turn on the option for the schema registry under environment on the right side.

    Stream Governance API option, create credentials, and save them locally
   ![Screenshot 2023-10-08 124932](https://github.com/Anirudh-Aravind/Real-Time-Data-Pipeline/assets/84184475/44915aed-78a0-43ac-b7da-50437b4c5078)

   Click on the Topics option under the cluster and create a topic
   ![image](https://github.com/Anirudh-Aravind/Real-Time-Data-Pipeline/assets/84184475/75324f2e-a308-4312-802b-2f39388fe2f2)


   Then create a schema according to our dataset by clicking the Schema option under the created topic.
   ![image](https://github.com/Anirudh-Aravind/Real-Time-Data-Pipeline/assets/84184475/e86ebbad-c588-4ec1-825f-d6883e61d0d5)

   There needs to set key and value, here both I am considering as Avro file format

     Here is how I defined the value
  
     {
    "fields": [
      {
        "name": "order_id",
        "type": "string"
      },
      {
        "name": "customer_id",
        "type": "string"
      },
      {
        "name": "order_status",
        "type": "string"
      },
      {
        "name": "order_purchase_timestamp",
        "type": "string"
      },
      {
        "default": null,
        "name": "order_approved_at",
        "type": [
          "null",
          "string"
        ]
      },
      {
        "default": null,
        "name": "order_delivered_carrier_date",
        "type": [
          "null",
          "string"
        ]
      },
      {
        "default": null,
        "name": "order_delivered_customer_date",
        "type": [
          "null",
          "string"
        ]
      },
      {
        "name": "order_estimated_delivery_date",
        "type": "string"
      }
    ],
    "name": "ecommerceorders",
    "namespace": "com.kaggle.ecommerceorders",
    "type": "record"
  }

    The same I uploaded as e-commerce.json

    The key I just simply mentioned as string


3. Kafka Producer:

     Use the Python Kafka producer code to read data from the pandas data frame and publish it to the 'ecommerce-orders' Kafka topic I previously defined. I combined the 'customer_id' and 'order_id' values from the dataset to produce the key for each message.

4. Apache Cassandra Setup:

   Configure and install Apache Cassandra. I used DataStax, a database platform that makes use of Apache Cassandra, for this.
   The DataStax URL is as follows: https://accounts.datastax.com/session-service/v1/login
  
   Create a database, and then in DataStax, establish a keyspace under it for storing the e-commerce data.
  
    To connect to your database of drivers, select the Connect option and download the Secure Connect Bundle.
   ![Screenshot 2023-10-08 131741](https://github.com/Anirudh-Aravind/Real-Time-Data-Pipeline/assets/84184475/d76189d1-6639-4112-bb5a-d541e98a083c)
  
   Select the Python language in the database's Drivers section. It demonstrates how to utilize a Python driver to link client applications to your Astra DB database.
   
   By selecting the Generate Token option, the CLIENT_ID & CLIENT_SECRET values used in the script may be obtained. Copy those values so that we can utilize them right away.


5. Cassandra Data Model:
   
   Created a table, named 'orders', within the 'ecommerce'
   keyspace. This table should reflect the schema of the incoming
   data and include additional columns for the derived features:
   'OrderHour' and 'OrderDayOfWeek'. The data model is having
   'customer_id' as the partition key and 'order_id' and
   'order_purchase_timestamp' as clustering keys.
 ![image](https://github.com/Anirudh-Aravind/Real-Time-Data-Pipeline/assets/84184475/956f25e3-4c23-4e63-9cd1-7c95a77a129e)
    
    Here is the DDL for table;
    
    CREATE TABLE ecommerce.orders (
                    order_id uuid,
                    customer_id uuid,
                    order_status text,
                    order_purchase_timestamp timestamp,
                    order_approved_at timestamp,
                    order_delivered_carrier_date timestamp,
                    order_delivered_customer_date timestamp,
                    order_estimated_delivery_date timestamp,
                    order_hour int,
                    Order_day_of_week text,
                    PRIMARY KEY ((customer_id), order_id,
                    order_purchase_timestamp));

6. Kafka Consumer and Data Transformation:
   
    Kafka consumer code (make sure to have a consumer group)
    in Python that subscribes to the 'ecommerce-orders' topic. The
    consumer is derived two new columns 'PurchaseHour' and
    'PurchaseDayOfWeek', then ingested transformed data into the
    'orders' table in Cassandra


   ![image](https://github.com/Anirudh-Aravind/Real-Time-Data-Pipeline/assets/84184475/63674afa-f0e5-44a3-aea6-fa061f0de22b)
   
   
   ![image](https://github.com/Anirudh-Aravind/Real-Time-Data-Pipeline/assets/84184475/51d5d0a2-da35-4f7c-83dc-2f0d6df544cb)

      





     





