# Lab 2a: Migrate MongoDB Applications to Azure Cosmos DB(Mongo API)
## Scenario 1 Understand the Flight Reservation App, Local MongoDB and test Application
### Understand Flight reservation application 
#### Access Jump VM 
1. From azure portal, Go to virtual machine and select the JumpVM. In Overview section, click on Connect button. It will show username with IP address copy that IP address.

   <img src="images/jumpvm1.jpg"/><br/>

1. Click on start button and search for **Remote Desktop Connection** and click on it.

1. Remote Desktop Connection window will pop-up in that provide the IP Address that you copied in above step.

<img src="images/rdc.jpg"/><br/>

1. Click on **Yes** button in Remote Desktop Connection Wizard.

<img src="images/rdc2.jpg"/><br/>

1. Enter the credentials provided in the mail to connect to VM.

<img src="images/xrdp.jpg"/><br/>

    > Note If you get err:connecting at first login: Click OK and try again.

#### Launch VS Code and understand application components and hierarchy 

1. Once you are logged in the Ubuntu OS click on Application for accessing visual studio code as shown blow:

<img src="images/jumpvm2.jpg"/><br/>

1. VS Code should have src folder already opened, Notice the folders hierarchy 

<img src="images/projects.png"/><br/>

1. ContosoAir application is two tiered application, One frontend application built on AngularJS named **ContosoAir.Website** and API application named **ContosoAir.Services** build with NodeJS. 

1. All connectivity information required for the API application is maintained in **config.js** , As shown below.
<img src="images/configjs.png"/><br/>

1. Following is the file system location for application binaries /home/CosmosDB-Hackfest/ContosoAir/src/

<img src="images/fsbrowse.png"/><br/>

#### Access Local MongoDB and Verify Collections

1. Start the mongodb Service by running **sudo service mongod start** and  verify the mongoDB database while running the mongo as shown below:

<img src="images/mongo.jpg"/><br/>

1. You can also verify the records in mongoDB database while running the following commands. Copy the db_name(contosoairdb3) for future use. Note down that there are 5 collections used by the Application
```bash
show dbs 
use <db_name>
show collections
```
<img src="images/mongodb.jpg"/><br/>

#### Execute and Test the Application Locally

1. To start the ContosoAir app service layer, go to /home/CosmosDB-Hackfest/ContosoAir/src/ContosoAir.Services in file system, right click and select **Open Terimal Here** as shown below.

<img src="images/jumpvm4.jpg"/><br/>


1. Run **npm start** command. Application should start, Incase of any errors make sure you're in right directory as instructed in previous step.Minmize the terminal once command is executed, do not exit or CTRL+C.


<img src="images/jumpvm5.jpg"/><br/>

1. Now, to start the ContosoAir Website layer, go to /home/CosmosDB-Hackfest/ContosoAir/src/ContosoAir.Website. Open terminal from there and run **ng serve** command in terminal. Minmize the terminal once command is executed, do not exit or CTRL+C.

<img src="images/jumpvm6.jpg"/><br/>

    > Note This  may take upto a minute to start this website. 

1. Copy the **localhost URL** [http://localhost:4200](http://localhost:4200) from the **terminal** and paste it in **Mozilla firefox** browser and press enter. There's a shortcut created for your already on desktop for this, you can open website using that as well.

1. You will see the sign-up page. You need to login to App using your Microsoft Account(formerly live-id). If you do not have a live-id, you can create one by following instructions given on page.


<img src="images/signup.jpg"/><br/>

1. Once you get login, you will be redirected to **ContosoAir app**.

<img src="images/contoso1.jpg"/><br/>

1. Enter **Departure date** and **Return date** in **YYYY-MM-DD** format and click **Find Flights** button.

<img src="images/contoso2.jpg"/><br/>
 


## Scenario 2 Migrate Application data to Cosmos DB

### Create Cosmos DB with Mongo API
1. Open Azure Portal with your credential and Click on Add button.

<img src="images/addcosmos.jpg"/><br/>

1. Search for Azure Cosmos DB and Select it.

<img src="images/cosmos1.jpg"/><br/>
1. Click on create button.

<img src="images/cosmos2.jpg"/><br/>
1. Populate the below parameters as shown below.
* **ID**:(any valid name)
* **API**: select **Mongo DB** from the dropdown.
* **Resource Group**: Choose use existing Resource Group.

<img src="images/cosmosdb1.jpg"/><br/>

1.	After deployment gets completed, click on **Go to resource** to verify that resource is successfully deployed.

<img src="images/cosmosdb2.jpg"/><br/>

1. **Go to Connection String** and Copy all the parameters (Host, Port, Username, Password, Primary Connection String) in notepad for future use
<img src="images/cosmosdb3.jpg"/><br/>

### Migrate Database to Azure Cosmos DB
In this exercise, We will use mongoexport to export the data from MongoDB running locally in the jump-VM and then we will use mongoimport to insert this dataset into newly created Cosmos DB.

1.	Open Terminal on Jump VM,(Alternatively you can also access Jump VM on SSH by any ssh tool such as Putty), verify that MongoD Service is running. Issue following commands to export the each collection in the mongo db database to a local bkp file. 

```bash
sudo mongoexport --host localhost --db contosoairdb3 --collection BookingsCollection --out BookingsCollection.bkp
sudo mongoexport --host localhost --db contosoairdb3 --collection DealsCollection --out DealsCollection.bkp
sudo mongoexport --host localhost --db contosoairdb3 --collection FlightsCollection --out FlightsCollection.bkp
sudo mongoexport --host localhost --db contosoairdb3 --collection SeatsCollection --out SeatsCollection.bkp
sudo mongoexport --host localhost --db contosoairdb3 --collection feedbackdb --out feedbackdb.bkp
```

 <img src="images/allexport.jpg"/><br/>

2.	Now, we will import this MongoDB on Azure Cosmos DB (MongoDB) and replace the HOST, PORT, USERNAME and PASSWORD with the parameters in below command with values you copied in above step.

```bash
mongoimport --host <HOST>:<PORT> -u <USERNAME> -p <PASSWORD> --ssl --sslAllowInvalidCertificates --db contoairdb3 --collection BookingsCollection --file BookingsCollection.bkp

mongoimport --host <HOST>:<PORT> -u <USERNAME> -p <PASSWORD> --ssl --sslAllowInvalidCertificates --db contoairdb3 --collection DealsCollection --file DealsCollection.bkp

mongoimport --host <HOST>:<PORT> -u <USERNAME> -p <PASSWORD> --ssl --sslAllowInvalidCertificates --db contoairdb3 --collection FlightsCollection --file FlightsCollection.bkp

mongoimport --host <HOST>:<PORT> -u <USERNAME> -p <PASSWORD> --ssl --sslAllowInvalidCertificates --db contoairdb3 --collection SeatsCollection --file SeatsCollection.bkp

mongoimport --host <HOST>:<PORT> -u <USERNAME> -p <PASSWORD> --ssl --sslAllowInvalidCertificates --db contoairdb3 --collection feedbackdb --file feedbackdb.bkp
```

 <img src="images/importall.jpg"/><br/>


1. Data is now migrated, lets verify this in Azure Portal, Switch to Azure Portal as launched earlier navigate to Resource groups option present in the favourites menu on the left side panel and select the resource group then click on your newly created Azure Cosmos DB Account.

1. Click on **Data Explorer** option. It will display the collection created in Azure Cosmos DB Account. You should see newly created DB along with collection, you may browse the documents in collections.

<img src="images/collections.jpg"/><br/>

### Update Application to use Cosmos DB 


1. Go back to Visual Studio Code IDE in Jump VM and enter Primary Connection String Against **MONGO_DB_COONECTION_STRING**(add database name contosoairdb3 before question mark in primary connection string) as in the config.js.

```bash
mongodb://cosmosdb12345:vMTETikja355VZjnJQGC3gwdLaR8xjNlpUq65loZVd4pLvmlG9PB25eqOb7V0EWFnvkqzd9GMp4vjiiDYGLahw==@cosmosdb12345.documents.azure.com:10255/**contosoairdb3**?ssl=true&replicaSet=globaldb
```

  <img src="images/config.jpg"/><br/>

1. Navigate back to the **Azure Portal** Resource groups option present in the favourites menu on the left side panel and select the resource group "" and click on **Azure Cosmos DB Account** then, click on **Replicate data globally** option present under **SETTINGS** section in Cosmos DB Account blade.

1. Copy the **WRITE REGION** and paste it against **DOCUMENT_DB_PREFERRED_REGION** key in **config.js** file which is already opened in Visual Studio Code IDE and save this file.

1. Open the terminal window running already running with **npm start** for Services application, Exit the application by pressing CTRL+C and start the application again by running **npm start**

  <img src="images/jumpvm4.jpg"/><br/>

  <img src="images/jumpvm5.jpg"/><br/>

1. ContosoAir Website should be running already in other terminal window, You can leave it running as is(No need to stop and restart).
if not running, you can manually start it to by going to /home/CosmosDB-Hackfest/ContosoAir/src/ContosoAir.Website. Open terminal from there and run **np serve** command in terminal.

<img src="images/jumpvm6.jpg"/><br/>

1. Copy the **localhost URL** [http://localhost:4200](http://localhost:4200) from the **terminal** and paste it in **Mozilla firefox** browser and press enter.

1. You will see the sign-up page. Enter your microsoft Credential here. 

<img src="images/signup.jpg"/><br/>

1. Once you get login, you will be redirected to **ContosoAir app**.

<img src="images/contoso1.jpg"/><br/>

1. Enter **Departure date** and **Return date** in **YYYY-MM-DD** format and click **Find Flights** button.

<img src="images/contosotime.jpg"/><br/>

You should notice that latency is now changed when loading the flight data

   > _Awesome, you have migrated your application database from local MongoDB instance to Azure Cosmos DB and your application is now using Cosmos DB.


# Cosmos DB Concepts

## Scenario 3: Partitioning

## Concept of Logical Partitioning

1. In **Azure Cosmos DB**, you can store and query schema-less data with order-of-millisecond response times at any scale.
1. Azure Cosmos DB provides **containers** for storing data called **collection.** **Containers** are logical resources and can span one or more physical partitions or servers.
1. The number of partitions is determined by Azure Cosmos DB based on the **storage** **size** and the provisioned **throughput** of the container. 
1. Partition management is fully managed by **Azure Cosmos DB**, and you don't have to write complex code or manage your partitions. **Azure Cosmos DB** containers are unlimited in terms of **storage** and **throughput.**
1. When you create a collection, you can specify a  **partition key** property. **Partition key** is the JSON property (or path) within your documents that can be used by **Mongo API** to distribute data among multiple partitions.
1. **Mongo API** will hash the partition key value and use the hashed result to determine the partition in which the JSON document will be stored. All documents with the same partition key will be stored in the same partition.

   >_In_ **Azure Cosmos DB,** logical partitioning _gets created based on the_ **size** _of the collection i.e. more than_ **10 GB** _or the specified **throughput of the container** (throughput in terms of [request units_](https://docs.microsoft.com/en-us/azure/cosmos-db/request-units) (RU) per second)._

   >_Physical partitioning_ **Partitioning** _is completely managed by_ **Azure**_. It automatically creates a_ **Logical Partition** _based on the_ **Partition key**. But in this case, just to show you the power of **Partitioning** feature of Azure Cosmos DB, we have created the **Physical partition.** 

Configure FlightsCollection data to leverage the Partitions

1. In this exercise, We'll create another collection named **FlightsCollectionPartitioned** in existing **contoairdb3** with number of **Stops** as the partition key
1. Open Azure Portal and access the Cosmos DB account created earlier, Go to **Browse** in settings blade

    ![](images/createcollection.jpg)

1. Click on Add Collection Button, Scroll horizontally to see the full dialogue box. Enter **contosoairdb3** as the database ID and **FlightsCollectionPartitioned** as the collection name. Enter **stop** as partition key, leave all other settings as default.

 ![](images/createnewcollection.jpg)

1. Now We'll insert the the **FlightsCollection** data into this partitioned collection. For this, We've already created a nodejs program named **cosmos_mongo_partition_insert.js** . run **node cosmos_mongo_partition_insert.js** the ContosoAir.Services root directory. Go to ContosoAir.Services by issuing **cd /home/dbadmin/CosmosDB-Hackfest/ContosoAir/src/ContosoAir.Services** You can use **ls** to verify the files in the directory

  ![](images/insertpartitiondata.jpg.jpg)

1. Data is now inserted, let's go to Azure portal to verify, Go to **Browse** in settings blade of Cosmos DB Account	
 

1. You'll see that under documents, there's another column named **stop** along with **id**. Click on Scale and settings to verify the partition. 

 ![](images/scalesettings.jpg)

1. In order to use this collection in the application, We need to edit **config.js** of Services application and replace following value with newly created collection. Save the file once changed.

    DOCUMENT_DB_FLIGHT:           'FlightsCollection',
replace with newly created collection with partitioning 
    DOCUMENT_DB_FLIGHT:           'FlightsCollectionPartitioned',
1. In order to test the application, exit the terminal running **npm start** for the application and start it again

> Awesome! In this scenario, you learned the Partitioning feature of  Azure Cosmos DB.

## Scenario 4: Global Distribution

To be added Later.

## Scenario 5: Integration with Azure Functions

### Create an Azure Function App
Login to Azure portal with your credentials and Click on +New
1. Login to Azure portal with your credentials and Click on +New
    ![](image/searchfunction.png)


1. Search for Function App

1. Click on Create

1. Fill in details to create Function App as defined below

•	App Name: Your unique name to identify the application
•	Subscription: Leave default
•	Resource Group: Choose Use Existing and select your available resource group
•	OS: Windows
•	Hosting Plan: Consumption Plan
•	Location: Location of your RG
•	Storage: Use Existing and choose available storage account

    ![](image/newfunction.png)

1. Click on Go to Resource Once provisioning is completed

    ![](image/functioncreated.png)



### Create a function with Deal Data

  > _Let's create an Azure Function to retrieve data of flight deals._

1. Go to **Resource groups** option ![](img/ResourceGropus.jpg)
   present in the favourites menu on the left side panel and select the resource group **<inject story-id="story://Content-Private/content/dfd/SP-GDA/gdaexpericence1/story_a_gda_using_cosmosdb" key="myResourceGroupName"/>** and click on Azure Function named **<inject story-id="story://Content-Private/content/dfd/SP-GDA/gdaexpericence1/story_a_gda_using_cosmosdb" key="azureFunctionName"/>**.
 
1. Hover over the **Functions** under **Functions Apps** and click on **+ sign** besides **Functions**.
  
1. Now, click on the **Custom function** link present at the bottom of the page.

    ![](image/GetStartedOnYourOwn.jpg)

1. Click on **C#** template available in **HTTP trigger** section and name the function as "**FetchDealsData**" in **Name** textbox and click **Create** button.
1. A function with sample default code would get created. Now, replace the sample default code with the code snippet given below.

    ```c#
    using System.Net;
    using System.Linq;

    public static HttpResponseMessage Run(HttpRequestMessage req, TraceWriter log, IEnumerable<dynamic> SelectDealsData)
    {
        return req.CreateResponse(HttpStatusCode.OK, SelectDealsData);
    }
    ```

    > **NOTE:**
    > In above code snippet, **IEnumerable&lt;dynamic> SelectDealsData** is a parameter used to fetch the list of **DealsData** collection from Cosmos DB. This method returns the result fetched from Cosmos DB along with **HttpStatusCode** with the help of **CreateResponse** method.

1. Click on **Save** button. 
1. Now, click on the **Integrate** option listed in **"FetchDealsData"** function in the **Function Apps** blade.

1. Under **Inputs** section, click on **+ New Input** and select **Azure Cosmos DB** then click **Select** button at the bottom of page.

    ![](image/Integrate.jpg)

1. Enter **Document parameter name** as **SelectDealsData**, **Database name** as **contosoairdb** and **Collection name** as **DealsCollection** in respective textboxes.
 
1. Copy the **SQL query** given below and paste it into **SQL Query (optional)** textbox.

    ```sql
   select c.id, c.fromName, c.fromCode, c.toName, c.toCode, c.price, c.departTime, c.arrivalTime, c.hours, c.stops, c.since from c
    ```

    > **NOTE:** Above **SQL query** is responsible to get deals details from **DealsCollection** available in Cosmos DB like id, from name, from code, to name, to code, price, depart time, arrival time, hours, stops, since. (Ignore non-mandatory fields)

    ![](image/SqlQuery.jpg)

1. To enter **Cosmos DB account connection,** click on the **new** link given beside **Cosmos DB account connection** textbox.
1. You will be redirected to **Cosmos DB Account blade**, select the **Cosmos DB Account** named as **<inject story-id="story://Content-Private/content/dfd/SP-GDA/gdaexpericence1/story_a_gda_using_cosmosdb" key="cosmosDBWithSQLDBName"/>**.
1. Now, click **Save** button ![](img/SaveButton.jpg) button.
1. To check whether the function is integrated, click on **FetchDealsData** function present under **AureFunctionForDeals** function app blade and click on **Test** option present at the right most corner.

   ![File2](image/file2.png)

1. Select **HTTP method** as **GET** from the dropdown. Then click **Run** button ![](img/run.jpg) at the bottom.
1. Status **200 Ok** will be displayed once the test is completed which signifies that the function is integrated successfully.

   ![](image/status200.jpg)

   > _Here you go! You have successfully created Azure Function to_ _retrieve flight deals_ _data from_ **Cosmos DB**_._

1. In the **Functions Apps** blade, click on **FetchDealsData**.
1. You will get navigated to function and will find **</> Get function URL** link in the top right corner of the page. Click on the link ![](image/getFunctn.jpg).
1. On clicking **</> Get function URL** you will get a popup window with a URL.

    ![](image/url.jpg)
1. Click on **Copy** ![](img/Copy.jpg) icon to copy the given URL and paste it against **AZURE\_FUNCTION\_DEALS\_URL:** variable in **config.js file** opened in **Visual Studio 2017** IDE and save the file.

   > _Awesome, you have created Azure Function for Deals data and integrated with Cosmos DB._


## Scenario 6: Azure Cosmos DB Operations: Monitoring, Security and Backup & Restore

## Understanding Cosmos DB Monitoring and SLA's

Cloud service offering a comprehensive SLA for:

1. **Availability:** The most classical SLA. Your system will be available for more than 99.99% of the time or you get a refund.
1. **Throughput:** At a collection level, the throughput for your database collection is always executed according to the maximum throughput you provisioned.
1. **Latency:** Since speed is important, 99% of your requests will have a latency below 10ms for document read or 15ms for document write operations.
1. **Consistency:** Consistency guarantees in accordance with the consistency levels chosen for your requests.

   >_Now let's walk through different SLA's and see how the data is displayed in graphical format._

1. Click on Azure Portal's **Resource Group** option present in the favourites blade in the left side panel and click on **<inject story-id="story://Content-Private/content/dfd/SP-GDA/gdaexpericence1/story_a_gda_using_cosmosdb" key="myResourceGroupName"/>.**
1. Click on **"<inject story-id="story://Content-Private/content/dfd/SP-GDA/gdaexpericence1/story_a_gda_using_cosmosdb" key="cosmosDBWithSQLDBName"/>"** which is your **Cosmos DB Account** and then click on **Metrics** option present under **Cosmos DB Account** blade.
1. SLA's based on parameters such as **Throughput, Storage, Availability, Latency and Consistency** will be viewed.

   ![](images/charts_thruput_menu.png)

#### Charts included under Throughput menu

1. Click on **Throughput** menu to view the graphs.
1. The first graph displayed is to find **Number of requests over the 1 hour period.**

   ![](images/Throughput1.jpg)

1. The second graph is to monitor **Number of requests that failed due to exceeding throughput or storage capacity provisioned for the collection per 5 min.**
1. The third chart displays **provisioned throughput and Max Request Units per second (RU/s) consumed by a physical partition, over a given 5-minute interval, across all partitions.**

    ![](images/Throughput2.jpg)

1. This chart **displays provisioned throughput and Max Request Units per second (RU/s) consumed by each physical partition over the last observed 1-minute interval.**

    ![](images/Throughput3.jpg)

1. The fourth chart shows the **Average Consumed Request Units per second (RU/s) over the 5-minute period vs provisioned RU/s.**

    ![](images/Throughput4.jpg.png)

    #### Charts included under Storage menu.

1. Click on **Storage** menu to view the graphs.
1. The first graph displays the **Available storage capacity for this collection, current size of the collection data, and index.**

    ![](images/data_index_storage_consume_vs_available.png)

1. Another graph shows the **Current size of data and index stored per physical partition.**

    ![](images/storage1.jpg)

1. The third chart shows the **Current number of documents stored per physical partition**.

    ![](images/storage2.jpg)

    #### Charts included under Availability menu.

1. Click on **Availability** menu to view the graphs.
1. The chart displays the **Availability is reported as % of successful request over the past hour, where successful is defined in the DocumentDB SLA.**

    ![](images/availability.jpg)

    #### Charts included under Latency menu.

1. Click on **Latency** menu to view the graphs.
1. First graph displays **the Latency for a 1KB document lookup operation observed in this account's read regions in the 99th percentile.**

    ![](images/latency1.jpg)

1. Another graph displays the Latency for a 1KB document write operation observed in South Central US in the 99th percentile.

    ![](images/latency2.jpg)

    #### Charts included under Consistency menu.

1. Click on **Consistency** menu to view the graphs.
1. The first graph displays **Empirical probability of a consistent read in read region as a function of time since the corresponding write commit in write region, observed over the past hour.**

    ![](images/empirical_portability.png)

1. This chart shows **distribution of the observed replication latency in {0}, i.e. the difference between time when the data write was committed in {1} and the time when the data became available for read in {0}.**

    ![](images/replication_latency.png)

1. The third graph displays **Percent of requests that met the monotonic read guarantee and the "read your own writes" guarantee.**

    ![](images/cosistency3.jpg)

    > For further information refer following link [Azure Cosmos DB  Service Level Agreements](https://azure.microsoft.com/en-us/support/legal/sla/cosmos-db/v1_1//).

>_Excellent job! Congratulations you successfully deep dived into the features of_ **Azure Cosmos DB** _._

## Securing Cosmos DB with Firewall
To secure data stored in an Azure Cosmos DB database account, Azure Cosmos DB has provided support for a secret based authorization model that utilizes a strong Hash-based message authentication code (HMAC). Now, in addition to the secret based authorization model, Azure Cosmos DB supports policy driven IP-based access controls for inbound firewall support. This model is similar to the firewall rules of a traditional database system and provides an additional level of security to the Azure Cosmos DB database account. With this model, you can now configure an Azure Cosmos DB database account to be accessible only from an approved set of machines and/or cloud services. Access to Azure Cosmos DB resources from these approved sets of machines and services still require the caller to present a valid authorization token.

#### IP Access Control Overview

By default, an Azure Cosmos DB database account is accessible from public internet as long as the request is accompanied by a valid authorization token. To configure IP policy-based access control, the user must provide the set of IP addresses or IP address ranges in CIDR form to be included as the allowed list of client IPs for a given database account. Once this configuration is applied, all requests originating from machines outside this allowed list are blocked by the server. The connection processing flow for the IP-based access control is described in the following diagram:

  ![](https://docs.microsoft.com/en-us/azure/cosmos-db/media/firewall-support/firewall-support-flow.png)
  
 To set the IP access control policy in the Azure portal, navigate to the Azure Cosmos DB account page, click **Firewall** in the navigation menu, then change the Enable IP Access Control value to **ON**. Once IP access control is on, the portal provides switches to enable access to the Azure portal, other Azure services, and the current IP.
    ![](https://docs.microsoft.com/en-us/azure/cosmos-db/media/firewall-support/azure-portal-firewall.png)

   ![](https://docs.microsoft.com/en-us/azure/cosmos-db/media/firewall-support/azure-portal-firewall-configure.png)
   
As a part of Lab, You may want to restrict access to your Database from the Linux VM where application is running and from Windows VM where PowerBI is running.


## Scale MongoDB Collections in Azure Cosmos DB

Azure Cosmos DB allows you to scale the throughput of a collection by just a couple of clicks.In order to scale up and down the throughput of a Cosmos DB collection, navigate to the Azure Cosmos DB account page and click on Scale.

Here you should see the current throughput and options to increase/decrease the throughput. You can make the necessary changes and Click **Save**.

    ![](images/scalethroughput.png)


## Understanding Cosmos DB Backup and restore functionality.

Azure Cosmos DB automatically takes backups of all your data at regular intervals. The automatic backups are taken without affecting the performance or availability of your database operations. All your backups are stored separately in another storage service, and those backups are globally replicated for resiliency against regional disasters. The automatic backups are intended for scenarios when you accidentally delete your Cosmos DB container and later require data recovery or a disaster recovery solution.
These automated backups are currently taken approximately every four hours and latest 2 backups are stored at all times. If the data is accidentally dropped or corrupted, please contact Azure support within eight hours.
Unlike your data that is stored inside Cosmos DB, the automatic backups are stored in Azure Blob Storage service. To guarantee the low latency/efficient upload, the snapshot of your backup is uploaded to an instance of Azure Blob storage in the same region as the current write region of your Cosmos DB database account. For resiliency against regional disaster, each snapshot of your backup data in Azure Blob Storage is again replicated via geo-redundant storage (GRS) to another region

### Backup retention period
As described above, Azure Cosmos DB takes snapshots of your data every four hours at the partition level. At any given time, only the last two snapshots are retained. However, if the collection/database is deleted, we retain the existing snapshots for all of the deleted partitions within the given collection/database for 30 days.

If you want to maintain your own snapshots, you can use the export to JSON option in the Azure Cosmos DB Data Migration tool to schedule additional backups.

### Restoring a database from an online backup
If you accidentally delete your database or collection, you can file a support ticket or call Azure support to restore the data from the last automatic backup. If you need to restore your database because of data corruption issue (includes cases where documents within a collection are deleted), see Handling data corruption as you need to take additional steps to prevent the corrupted data from overwriting the existing backups. For a specific snapshot of your backup to be restored, Cosmos DB requires that the data was available for the duration of the backup cycle for that snapshot.

### Handling data corruption
Azure Cosmos DB retains the last two backups of every partition in the database account. This model works well when a container (collection of documents, graph, table) or a database is accidentally deleted since one of the last versions can be restored. However, in the case when users may introduce a data corruption issue, Azure Cosmos DB may be unaware of the data corruption, and it is possible that the corruption may have overwritten the existing backups. As soon as corruption is detected, the user should delete the corrupted container (collection/graph/table) so that backups are protected from being overwritten with corrupted data.


See this for more inforamtion on Cosmos DB Backup and recovery: https://docs.microsoft.com/en-us/azure/cosmos-db/online-backup-and-restore


