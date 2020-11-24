- [Lab 3: Migrate Cassandra Workloads to Cosmos DB](#lab-3-migrate-cassandra-workloads-to-cosmos-db)
  - [Exercise 1: Setup](#exercise-1-setup)
    - [Task 1: Create a Resource Group and Virtual Network](#task-1-create-a-resource-group-and-virtual-network)
    - [Task 2: Create a Cassandra Database Server](#task-2-create-a-cassandra-database-server)
    - [Task 3: Populate the Cassandra Database](#task-3-populate-the-cassandra-database)
  - [Exercise 2: Migrate Data from Cassandra to Cosmos DB Using the CQLSH COPY Command](#exercise-2-migrate-data-from-cassandra-to-cosmos-db-using-the-cqlsh-copy-command)
    - [Task 1: Create a Cosmos Account and Database](#task-1-create-a-cosmos-account-and-database)
    - [Task 2: Export the Data from the Cassandra Database](#task-2-export-the-data-from-the-cassandra-database)
    - [Task 3: Import the Data to Cosmos DB](#task-3-import-the-data-to-cosmos-db)
    - [Task 4: Verify that Data Migration was Successful](#task-4-verify-that-data-migration-was-successful)
    - [Task 5: Clean Up](#task-5-clean-up)
  - [Exercise 3: Migrate Data from Cassandra to Cosmos DB Using Spark](#exercise-3-migrate-data-from-cassandra-to-cosmos-db-using-spark)
    - [Task 1: Create a Spark Cluster](#task-1-create-a-spark-cluster)
    - [Task 2: Create a Notebook for Migrating Data](#task-2-create-a-notebook-for-migrating-data)
    - [Task 3: Connect to Cosmos DB and Create Tables](#task-3-connect-to-cosmos-db-and-create-tables)
    - [Task 4: Connect to the Cassandra Database and Retrieve Data](#task-4-connect-to-the-cassandra-database-and-retrieve-data)
    - [Task 5: Insert Data into Cosmos DB Tables and Run the Notebook](#task-5-insert-data-into-cosmos-db-tables-and-run-the-notebook)
    - [Task 6: Verify that Data Migration was Successful](#task-6-verify-that-data-migration-was-successful)
    - [Task 7: Clean Up](#task-7-clean-up)

# Lab 3: Migrate Cassandra Workloads to Cosmos DB

In this lab, you'll migrate two datasets from Cassandra to Cosmos DB. You'll move the data in two ways. First, you'll export the data from Cassandra and use the **CQLSH COPY** command to import the database into Cosmos DB. Then, you'll migrate the data using Spark. You'll verify that migration was successful by running queries against the data held in the Cosmos DB database. 

The scenario for this lab concerns an ecommerce system. Customers can place orders for goods. The customer and order details are recorded in a Cassandra database. 

The lab runs using the Azure Cloud shell and the Azure portal.

## Exercise 1: Setup

In the first exercise, you'll create the Cassandra database for holding the customer and order data.

### Task 1: Create a Resource Group and Virtual Network

1. In your Internet browser, navigate to https://portal.azure.com and sign in.
1. In the Azure portal, click **Resource groups**, and then click **+Add**.
1. On the **Create a resource group page**, enter the following details, and then click **Review + create**:

    | Property  | Value  |
    |---|---|
    | Subscription | *\<your-subscription\>* |
    | Resource Group | cassandradbrg |
    | Region | Select your nearest location |

1. Click **Create**, and wait for the resource group to be created.
1. In the left-hand pane of the Azure portal, click **+ Create a resource**.
1. On the **New** page, in the **Search the Marketplace** box, type **Virtual Network**, and press Enter.
1. On the **Virtual Network** page, click **Create**.
1. On the **Basics** page, enter the following details, and then click **Next:IP Addresses**:

    | Property  | Value  |
    |---|---|
    | Subscription | *\<your-subscription\>* |
    | Resource Group | cassandradbrg |
    | Name | databasevnet |
    | Region | Select the same location that you specified for the resource group |

1. On the **IP Addresses** page, set the **IPv4 address space** to **10.0.0.0/24**.
1. Select the default subnet, and then select **Remove subnet**.
1. Select **+ Add subnet**. In the **Add subnet** pane, set the **Subnet name** to **default**, set the **Subnet address range** to **10.0.0.0/28**, and then click **Add**.
1. On the **IP Addresses** page, click **Next: Security**.
1. On the **Security** page, enter the following details, and then click **Review + create**:

    | Property  | Value  |
    |---|---|
    | BastionHost | Disabled |
    | DDoS Protection Standard  | Disabled |
    | Firewall | Disabled |

1. On the **Review + create** page, click **Create** and then wait for the virtual network to be created before continuing.

### Task 2: Create a Cassandra Database Server

1. In the left-hand pane of the Azure portal, click **+ Create a resource**.
1. In the **Search the Marketplace** box, type **Cassandra Certified by Bitnami**, and then press Enter.
1. On the **Cassandra Certified by Bitnami** page, click **Create**.
1. On the **Create a virtual machine** page, enter the following details, and then click **Next: Disks \>**.

    | Property  | Value  |
    |---|---|
    | Subscription | *\<your-subscription\>* |
    | Resource Group | cassandradbrg |
    | Virtual machine name | cassandraserver |
    | Region | Select the same location that you specified for the resource group |
    | Availability options | No infrastructure redundancy required |
    | Image | Cassandra Certified by Bitnami - Gen1 |
    | Size | Standard D2 v2 |
    | Authentication type | Password |
    | Username | azureuser |
    | Password | Pa55w.rdPa55w.rd |
    | Confirm password | Pa55w.rdPa55w.rd |

1. On the **Disks** page, accept the default settings, and then click **Next: Networking \>**.
1. On the **Networking** page, enter the following details, and then click **Next: Management \>**.

    | Property  | Value  |
    |---|---|
    | Virtual network | databasevnet |
    | Subnet | default (10.0.0.0/28) |
    | Public IP | (new) cassandraserver-ip |
    | NIC network security group | Advanced |
    | Configure network security group | (new) cassandraserver-nsg |
    | Accelerated networking | Off |
    | Load balancing | No |

1. On the **Management** page, accept the default settings, and then click **Next: Advanced \>**.
1. On the **Advanced** page, accept the default settings, and then click **Next: Tags \>**.
1. On the **Tags** page, accept the default settings, and then click **Next: Review + create \>**.
1. On the validation page, click **Create**.
1. Wait for the virtual machine to be deployed before continuing.
1. In the left-hand pane of the Azure portal, click **All resources**.
1. On the **All resources** page, click **cassandraserver-nsg**.
1. On the **cassandraserver-nsg** page, under **Settings**, click **Inbound security rules**.
1. On the **cassandraserver-nsg - Inbound security rules** page, click **+ Add**.
1. In the **Add inbound security rule** pane, enter the following details, and then click **Add**:

    | Property  | Value  |
    |---|---|
    | Source | Any |
    | Source port ranges | * |
    | Destination | Any |
    | Destination port ranges | 9042 |
    | Protocol | Any |
    | Action | Allow |
    | Priority | 1020 |
    | Name | Cassandra-port |
    | Description | Port that clients use to connect to Cassandra |

### Task 3: Populate the Cassandra Database

1. In the left-hand pane of the Azure portal, click **All resources**.
1. On the **All resources** page, click **cassandraserver-ip**.
1. On the **cassandraserver-ip** page, make a note of the **IP address**.
1. In the toolbar at the top of the Azure portal, click **Cloud Shell**.
1. If the **You have no storage mounted** message box appears, click **Create storage**.
1. When the Cloud Shell starts, in the drop-down list above the Cloud Shell window, select **Bash**.
1. In the Cloud Shell, if you haven't performed Lab 2, run the following command to download the sample code and data for this workshop:

    ```bash
    git clone https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB migration-workshop-apps
    ```

1. Move to the **migration-workshop-apps/Cassandra** folder:

    ```bash
    cd ~/migration-workshop-apps/Cassandra
    ```

1. Enter the following commands to copy the setup scripts and data to the **cassandraserver** virtual machine. Replace *\<ip address\>* with the value of the **cassandraserver-ip** IP address:

    ```bash
    scp *.* azureuser@<ip address>:~
    ```

1. At the prompt, type **yes** to continue connecting.
1. At the **Password** prompt, enter the password **Pa55w.rdPa55w.rd**
1. Type the following command to connect to the **cassandraserver** virtual machine. Specify the IP address of the **cassandraserver** virtual machine:

    ```bash
    ssh azureuser@<ip address>
    ```

1. At the **Password** prompt, enter the password **Pa55w.rdPa55w.rd**
1. Run the following command to connect to the Cassandra database, create the tables required by this lab, and populate them.

    ```bash
    bash upload.sh
    ```

    The script creates two keyspaces named **customerinfo** and **orderinfo**. The script creates a table named **customerdetails** in the **customerinfo** keyspace, and two tables named **orderdetails** and **orderline** in the**orderinfo** keyspace.

1. Run the following command, and make a note of the default password in this file:

    ```bash
    cat bitnami_credentials
    ```

1. Start the Cassandra Query Shell as the user **cassandra** (this is the name of the default Cassandra user created when the virtual machine was set up). Replace *\<password\>* with the default password from the previous step:

    ```bash
    cqlsh -u cassandra -p <password>
    ```

1. At the **cassandra@cqlsh** prompt, run the following command. This command displays the first 100 rows from the **customerinfo.customerdetails** table:

    ```cqlsh
    select *
    from customerinfo.customerdetails
    limit 100;
    ```

    Note that the data is clustered by the **stateprovince** column, and then ordered by **customerid**. This grouping enables applications to quickly find all customers located in the same region.

1. Run the following command. This command displays the first 100 rows from the **orderinfo.orderdetails** table:

    ```cqlsh
    select *
    from orderinfo.orderdetails
    limit 100;
    ```

    The **orderinfo.orderdetails** table contains a list of orders placed by each customer. The data recorded includes the date the order was placed, and the value of the order. The data is clustered by the **customerid** column, so that applications can quickly find all orders for a specified customer.

1. Run the following command. This command displays the first 100 rows from the **orderinfo.orderline** table:

    ```cqlsh
    select *
    from orderinfo.orderline
    limit 100;
    ```

    This table contains the items for each order. The data is clustered by the **orderid** column, and sorted by **orderline**.

1. Quit the Cassandra Query Shell:

    ```cqlsh
    exit;
    ```

1. At the **bitnami@cassandraserver** prompt, type the following command to disconnect from the Cassandra server and return to the Cloud Shell:

    ```bash
    exit
    ```

## Exercise 2: Migrate Data from Cassandra to Cosmos DB Using the CQLSH COPY Command 

You have now created and populated a Cassandra database. In this exercise, you will create a Cosmos DB account using the Cassandra API, and then migrate the data from your Cassandra database to a database in the Cosmos DB account.

### Task 1: Create a Cosmos Account and Database

1. Return to the Azure portal.
1. In the left pane, click **+ Create a resource**.
1. On the **New** page, in the **Search the Marketplace** box, type **Azure Cosmos DB**, end then press Enter.
1. On the **Azure Cosmos DB** page, click **Create**.
1. On the **Create Azure Cosmos DB Account** page, enter the following settings, and then click **Review + create**:

    | Property  | Value  |
    |---|---|
    | Subscription | Select your subscription |
    | Resource group | cassandradbrg |
    | Account Name | cassandra*nnn*, where *nnn* is a random number selected by you |
    | API | Cassandra |
    | Notebooks | Off |
    | Location | Specify the same location that you used for the Cassandra server and virtual network |
    | Capacity mode | Provisioned throughput |
    | Apply Free Tier Discount | Apply |
    | Account Type | Non-Production |
    | Geo-Redundancy | Disable |
    | Multi-region Writes | Disable |
    | Availability Zones | Disable |

1. On the validation page, click **Create**, and wait for the Cosmos DB account to be deployed.
1. In the left-hand pane, click **All resources**.
1. On the **All resources** page, click your Cosmos DB account (**cassandra*nnn***).
1. On the **cassandra*nnn*** page, click **Data Explorer**.
1. In the **Data Explorer** pane, click **New Table**.
1. In the **Add Table** pane, specify the following settings, and then click **OK**:

    | Property  | Value  |
    |---|---|
    | Keyspace name | Click **Create new**, and then type **customerinfo** |
    | Provision keyspace throughput | de-selected |
    | Enter tableId | customerdetails |
    | *CREATE TABLE* box | (customerid int, firstname text, lastname text, email text, stateprovince text, PRIMARY KEY ((stateprovince), customerid)) |
    | Throughput | 10000 |

1. In the **Data Explorer** pane, click **New Table**.
1. In the **Add Table** pane, specify the following settings, and then click **OK**:

    | Property  | Value  |
    |---|---|
    | Keyspace name | Click **Create new**, and then type **orderinfo** |
    | Provision keyspace throughput | de-selected |
    | Enter tableId | orderdetails |
    | *CREATE TABLE* box | (orderid int, customerid int, orderdate date, ordervalue decimal, PRIMARY KEY ((customerid), orderdate, orderid)) |
    | Throughput | 10000 |

1. In the **Data Explorer** pane, click **New Table**.
1. In the **Add Table** pane, specify the following settings, and then click **OK**:

    | Property  | Value  |
    |---|---|
    | Keyspace name | Click **Use existing**, and then select **orderinfo** |
    | Enter tableId | orderline |
    | *CREATE TABLE* box | (orderid int, orderline int, productname text, quantity smallint, orderlinecost decimal, PRIMARY KEY ((orderid), productname, orderline)) |
    | Throughput | 10000 |

### Task 2: Export the Data from the Cassandra Database

1. Return to the Cloud Shell.
1. Run the following command to connect to the cassandra server. Replace *\<ip address\>* with the IP address of the virtual machine. Enter the password **Pa55w.rdPa55w.rd** when prompted:

    ```bash
    ssh azureuser@<ip address>
    ```

1. Start the Cassandra Query Shell. Specify the password from the **bitnami_credentials** file:

    ```bash
    cqlsh -u cassandra -p <password>
    ```

1. At the **cassandra@cqlsh** prompt, run the following command. This command downloads the data in the **customerinfo.customerdetails** table and writes it to a file named **customerdata**. The command should export 19119 rows:

    ```cqlsh
    copy customerinfo.customerdetails
    to 'customerdata';
    ```

1. Run the following command to export the data in the **orderinfo.orderdetails** table to a file named **orderdata**. This command should export 31465 rows:

   ```cqlsh
    copy orderinfo.orderdetails
    to 'orderdata';
    ```

1. Run the following command to export the data in the **orderinfo.orderline** table to a file named **orderline**. This command should export 121317 rows:

   ```cqlsh
    copy orderinfo.orderline
    to 'orderline';
    ```

1. Close the Cassandra Query Shell:

    ```cqlsh
    exit;
    ```

### Task 3: Import the Data to Cosmos DB

1. Switch back to your Cosmos DB account in the Azure portal.
1. Under **Settings**, click **Connection String**, and make a note of the following items:

   - Contact Point
   - Port
   - Username
   - Primary Password

1. Return to the Cassandra server, and start the Cassandra Query Shell. This time, connect to your Cosmos DB account. Replace the arguments to the **cqlsh** command with the values you just noted:

    ```bash
    export SSL_VERSION=TLSv1_2
    export SSL_VALIDATE=false

    cqlsh <contact point> <port> -u <username> -p <primary password> --ssl
    ```

    Note that Cosmos DB requires an SSL connection.

1. At the **cqlsh** prompt, run the following commands to import the data that you previously exported from the Cassandra database:

    ```cqlsh
    copy customerinfo.customerdetails from 'customerdata' with chunksize = 200;
    copy orderinfo.orderdetails from 'orderdata' with chunksize = 200;
    copy orderinfo.orderline from 'orderline' with chunksize = 100;
    ```

    These commands should import 19119 **customerdetails** rows, 31465 **orderdetails** rows, and 121317 **orderline** rows. If these commands fail with a timeout error, you can resolve the problem either by:
    - Increasing the throughput for the corresponding table in the Azure portal (and reducing it again afterwards, to avoid excessive throughput charges), or
    - Lowering the chunk size of each copy operation. Decreasing the chunk size slows the ingestion rate, whereas increasing the throughput raises the costs.

### Task 4: Verify that Data Migration was Successful

1. Return to your Cosmos DB account in Azure portal, and then click **Data Explorer**.
1. In the **Data Explorer** pane, expand the **customerinfo** keyspace, expand the **customerdetails** table, and then click **Rows**. Verify that a set of customers appears.
1. Click **Add new clause**.
1. In the **Field** box, select **stateprovince**, and in the **Value** box, type **Tasmania**.
1. In the toolbar, click **Run Query**. Verify that the query returns 106 rows.
1. In the **Data Explorer** pane, expand the **orderinfo** keyspace, expand the **orderdetails** table, and then click **Rows**.
1. Click **Add new clause**.
1. In the **Field** box, select **customerid**, and in the **Value** box, type **13999**.
1. In the toolbar, click **Run Query**. Verify that the query returns 2 rows. Note the **orderid** for the first row (it should be 46899).
1. In the **Data Explorer** pane, expand the **orderinfo** keyspace, expand the **orderline** table, and then click **Rows**.
1. Click **Add new clause**.
1. In the **Field** box, select **orderid**, and in the **Value** box, type **46899**.
1. In the toolbar, click **Run Query**. Verify that the query returns 1 row, listing the product being ordered as **Road-550-W Yellow, 38**.

You have successfully migrated a Cassandra database to Cosmos DB by using the CQLSH COPY command.

### Task 5: Clean Up

1. Switch back to the Cassandra Query Shell running on your Cassandra server. The shell should still be connected to your Cosmos DB account. If you closed the shell earlier, then reopen it, as follows:

    ```bash
    export SSL_VERSION=TLSv1_2
    export SSL_VALIDATE=false

    cqlsh <contact point> <port> -u <username> -p <primary password> --ssl
    ```

1. In the Cassandra Query Shell, run the following commands to remove the keyspaces (and tables):

    ```cqlsh
    drop keyspace customerinfo;
    drop keyspace orderinfo;
    exit;
    ```

## Exercise 3: Migrate Data from Cassandra to Cosmos DB Using Spark

In this exercise, you'll migrate the same data used previously, but this time you'll use Spark from an Azure Databricks notebook.

### Task 1: Create a Spark Cluster

1. In the Azure portal, in the left-hand pane, click **+ Create a resource**.
1. In the **New** pane, in the **Search the Marketplace** box, type **Azure Databricks**, and then press Enter.
1. On the **Azure Databricks** page, click **Create**.
1. On the **Azure Databricks Service** page, enter the following details, and then click **Review + create**:

    | Property  | Value  |
    |---|---|
    | Subscription | *\<your-subscription\>* |
    | Resource Group | Use existing, cassandradbrg |
    | Workspace name | CassandraMigration |
    | Location | Select the same location that you specified for the resource group |
    | Pricing Tier | Standard |

1. On the **Review + create** page, click **Create** and then wait for the Databricks Service to be deployed.
1. In the left-hand pane, click **Resource groups**, click **cassandradbrg**, and then click the **CassandraMigration** Databricks Service.
1. On the **CassandraMigration** page, click **Launch Workspace**.
1. On the **Azure Databricks** page, under **Common Tasks**, click **New Cluster**.
1. On the **New Cluster** page, enter the following settings, and then click **Create Cluster**:

    | Property  | Value  |
    |---|---|
    | Cluster Name | MigrationCluster |
    | Cluster Mode | Standard |
    | Pool | None |
    | Databrick Runtime Version | Runtime: 5.5 LTS (Scala 2.11, Spark 2.4.3) |
    | Python Version | 3 |
    | Enable autoscaling | Selected |
    | Terminate after | 60 |
    | Worker Type | Accept the default settings |
    | Driver Type | Same as worker |

1. Wait for the cluster to be created; the state of the **MigrationCluster** is reported as **Running** when the cluster is ready. This process will take several minutes.

### Task 2: Create a Notebook for Migrating Data

1. In the pane to the left, click **Clusters**, click the **Libraries** tab, and then click **Install New**.
1. In the **Install Library** dialog, enter the following settings, and then click **Install**:

    | Property  | Value  |
    |---|---|
    | Library Source | Maven |
    | Coordinates | com.datastax.spark:spark-cassandra-connector_2.11:2.4.3 |
    | Repository | Leave blank |
    | Exclusions | Leave blank |

    This library contains the classes for connecting to Cassandra from Spark.

1. When the connector library is installed, click **Install Library**.
1. In the **Install Library** dialog, enter the following settings, and then click **Install**:

    | Property  | Value  |
    |---|---|
    | Library Source | Maven |
    | Coordinates | com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.2.0 |
    | Repository | Leave blank |
    | Exclusions | Leave blank |

    This library contains the classes for connecting to Cosmos DB from Spark.

<!--1. In the pane to the left of the **Cluster** page, click the **Azure Databricks** icon at the top.
1. On the **Azure Databricks** page, under **Common Tasks**, click **Import Library**.
1. On the **Create Library** page, enter the following settings, and then click **Create**:

    | Property  | Value  |
    |---|---|
    | Library Source | Maven |
    | Coordinates | com.datastax.spark:spark-cassandra-connector_2.11:2.4.3 |
    | Repository | Leave blank |
    | Exclusions | Leave blank |

    This library contains the classes for connecting to Cassandra from Spark.

1. When the **Status on running clusters** section appears, select the *check box adjacent to **Not installed** in the **MigrationCluster** row, and then click **Install**.
1. Wait until the status of the library changes to **Installed** before continuing.
1. In the pane to the left, click **Azure Databricks**.
1. On the **Azure Databricks** page, under **Common Tasks**, click **Import Library** again.
1. On the **Create Library** page, enter the following settings, and then click **Create**:

    | Property  | Value  |
    |---|---|
    | Library Source | Maven |
    | Coordinates | com.microsoft.azure.cosmosdb:azure-cosmos-cassandra-spark-helper:1.2.0 |
    | Repository | Leave blank |
    | Exclusions | Leave blank |

    This library contains the classes for connecting to Cosmos DB from Spark.

1. When the **Status on running clusters** section appears, select the check box adjacent to **Not installed** in the **MigrationCluster** row, and then click **Install**.
1. Wait until the status of the library changes to **Installed** before continuing.-->
1. In the pane to the left, click **Azure Databricks**.
1. On the **Azure Databricks** page, under **Common Tasks**, click **New Notebook**.
1. In the **Create Notebook** dialog box, enter the following settings, and then click **Create**:

    | Property  | Value  |
    |---|---|
    | Name | MigrateData |
    | Language | Scala |
    | Cluster | MigrationCluster |

### Task 3: Connect to Cosmos DB and Create Tables

1. In the first cell of the notebook, enter the following code:

    ```scala
    // Import libraries

    import org.apache.spark.sql.cassandra._
    import org.apache.spark.sql._
    import org.apache.spark._
    import com.datastax.spark.connector._
    import com.datastax.spark.connector.cql.CassandraConnector
    import com.microsoft.azure.cosmosdb.cassandra
    ```

    This code imports the types required to connect to Cosmos DB and Cassandra from Spark.

1. In the toolbar on the right of the cell, click the drop-down arrow, and then click **Add Cell Below**.
1. In the new cell, enter the following code. Specify the Contact Point, Username, and Primary Password with the values for your Cosmos DB account (you recorded these values in the previous exercise):

    ```scala
    // Configure connection parameters for Cosmos DB

    val cosmosDBConf = new SparkConf()
        .set("spark.cassandra.connection.host", "<contact point>")
        .set("spark.cassandra.connection.port", "10350")
        .set("spark.cassandra.connection.ssl.enabled", "true")
        .set("spark.cassandra.auth.username", "<username>")
        .set("spark.cassandra.auth.password", "<primary password>")
        .set("spark.cassandra.connection.factory",
            "com.microsoft.azure.cosmosdb.cassandra.CosmosDbConnectionFactory")
        .set("spark.cassandra.output.batch.size.rows", "1")
        .set("spark.cassandra.connection.connections_per_executor_max", "1")
        .set("spark.cassandra.output.concurrent.writes", "1")
        .set("spark.cassandra.concurrent.reads", "1")
        .set("spark.cassandra.output.batch.grouping.buffer.size", "1")
        .set("spark.cassandra.connection.keep_alive_ms", "600000000")
    ```

    This code sets the Spark session parameters to connect to your Cosmos DB account

1. Add another cell below the current one, and enter the following code:

    ```scala
    // Create keyspaces and tables

    val cosmosDBConnector = CassandraConnector(cosmosDBConf)

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE KEYSPACE customerinfo WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1}"))
    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE customerinfo.customerdetails (customerid int, firstname text, lastname text, email text, stateprovince text, PRIMARY KEY ((stateprovince), customerid)) WITH cosmosdb_provisioned_throughput=10000"))

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE KEYSPACE orderinfo WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1}"))
    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE orderinfo.orderdetails (orderid int, customerid int, orderdate date, ordervalue decimal, PRIMARY KEY ((customerid), orderdate, orderid)) WITH cosmosdb_provisioned_throughput=10000"))

    cosmosDBConnector.withSessionDo(session => session.execute("CREATE TABLE orderinfo.orderline (orderid int, orderline int, productname text, quantity smallint, orderlinecost decimal, PRIMARY KEY ((orderid), productname, orderline)) WITH cosmosdb_provisioned_throughput=10000"))
    ```

    These statements rebuild the orderinfo and customerinfo keyspaces, together with the tables. Each table is provisioned with 10000 RU/s of throughput.

### Task 4: Connect to the Cassandra Database and Retrieve Data

1. In the notebook, add another cell, and enter the following code. Replace *\<ip address\>* with the IP address of the virtual machine, and specify the password you retrieved earlier from the **bitnami_credentials** file:

    ```scala
    // Configure connection parameters for the source Cassandra database

    val cassandraDBConf = new SparkConf()
        .set("spark.cassandra.connection.host", "<ip address>")
        .set("spark.cassandra.connection.port", "9042")
        .set("spark.cassandra.connection.ssl.enabled", "false")
        .set("spark.cassandra.auth.username", "cassandra")
        .set("spark.cassandra.auth.password", "<password>")
        .set("spark.cassandra.connection.connections_per_executor_max", "10")
        .set("spark.cassandra.concurrent.reads", "512")
        .set("spark.cassandra.connection.keep_alive_ms", "600000000")
    ```

1. Add another cell, and enter the following code:

    ```scala
    // Retrieve the customer and order data from the source database

    val cassandraDBConnector = CassandraConnector(cassandraDBConf)
    var cassandraSparkSession = SparkSession
        .builder()
        .config(cassandraDBConf)
        .getOrCreate()

    val customerDataframe = cassandraSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "customerdetails", "keyspace" -> "customerinfo"))
        .load

    println("Read " + customerDataframe.count + " rows")

    val orderDetailsDataframe = cassandraSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderdetails", "keyspace" -> "orderinfo"))
        .load

    println("Read " + orderDetailsDataframe.count + " rows")

    val orderLineDataframe = cassandraSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderline", "keyspace" -> "orderinfo"))
        .load

    println("Read " + orderLineDataframe.count + " rows")
    ```

    This block of code retrieves the data from the tables in the Cassandra database into Spark DataFrame objects. The code displays the number of rows read from each table.

### Task 5: Insert Data into Cosmos DB Tables and Run the Notebook

1. Add a final cell, and enter the following code:

    ```scala
    // Write the customer data to Cosmos DB

    val cosmosDBSparkSession = SparkSession
        .builder()
        .config(cosmosDBConf)
        .getOrCreate()

    // Connect to the existing table from Cosmos DB
    var customerCopyDataframe = cosmosDBSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "customerdetails", "keyspace" -> "customerinfo"))
        .load

    // Merge the results from the Cassandra database into the DataFrame
    customerCopyDataframe = customerCopyDataframe.union(customerDataframe)

    // Write the results back to Cosmos DB
    customerCopyDataframe.write
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "customerdetails", "keyspace" -> "customerinfo"))
        .mode(org.apache.spark.sql.SaveMode.Append)
        .save()

    // Write the order data to Cosmos DB, using the same strategy
    var orderDetailsCopyDataframe = cosmosDBSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderdetails", "keyspace" -> "orderinfo"))
        .load

    orderDetailsCopyDataframe = orderDetailsCopyDataframe.union(orderDetailsDataframe)

    orderDetailsCopyDataframe.write
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderdetails", "keyspace" -> "orderinfo"))
        .mode(org.apache.spark.sql.SaveMode.Append)
        .save()

    var orderLineCopyDataframe = cosmosDBSparkSession
        .read
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderline", "keyspace" -> "orderinfo"))
        .load

    orderLineCopyDataframe = orderLineCopyDataframe.union(orderLineDataframe)

    orderLineCopyDataframe.write
        .format("org.apache.spark.sql.cassandra")
        .options(Map( "table" -> "orderline", "keyspace" -> "orderinfo"))
        .mode(org.apache.spark.sql.SaveMode.Append)
        .save()
    ```

    This code creates another DataFrame for each of the tables in the Cosmos DB database. Each DataFrame will be empty, initially. The code then uses the **union** function to append the data from the corresponding DataFrame for each of the Cassandra tables. Finally, the code writes the appended DataFrame back to the Cosmos DB table.

    The DataFrame API is a very powerful abstraction provided by Spark, and is a highly efficient structure for transporting large volumes of data very quickly.

1. In the toolbar at the top of the notebook, click **Run All**.  You will see messages indicating that the cluster is starting up. When the cluster is ready, the notebook runs the code in each cell in turn. You will see further messages appearing below each cell. The data transfer operations that read and write DataFrames are executed as Spark jobs. You can expand the job to view the progress. The code in each cell should complete successfully, without displaying any error messages.

### Task 6: Verify that Data Migration was Successful

1. Return to your Cosmos DB account in the Azure portal.
1. Click **Data Explorer**,
1. In the **Data Explorer** pane, expand the **customerinfo** keyspace, expand the **customerdetails** table, and then click **Rows**. The first 100 rows should be displayed. If the keyspace does not appear in the **Data Explorer** pane, click **Refresh** to update the display.
1. Expand the **orderinfo** keyspace, expand the **orderdetails** table, and then click **Rows**. The first 100 rows should be displayed for this table as well.
1. Finally, expand the **orderline** table, and then click **Rows**. Verify that the first 100 rows for this table appear.

You have successfully migrated a Cassandra database to Cosmos DB by using Spark from a Databricks notebook.

### Task 7: Clean Up

1. In the Azure portal, in the left-hand pane, click **Resource groups**.
1. In the **Resource groups** window, click **cassandradbrg**.
1. Click **Delete resource group**.
1. On the **Are you sure you want to delete "cassandradbrg"** page, in the **Type the resource group name** box, enter **cassandradbrg**, and then click **Delete**.

---
© 2019 Microsoft Corporation. All rights reserved.

The text in this document is available under the [Creative Commons Attribution 3.0
License](https://creativecommons.org/licenses/by/3.0/legalcode), additional terms may apply. All other content contained in this document (including, without limitation, trademarks, logos, images, etc.) are
**not** included within the Creative Commons license grant. This document does not provide you with any legal rights to any intellectual property in any Microsoft product. You may copy and use this document for your internal, reference purposes.

This document is provided "as-is." Information and views expressed in this document, including URL and other Internet Web site references, may change without notice. You bear the risk of using it. Some examples are for illustration only and are fictitious. No real association is intended or inferred. Microsoft makes no warranties, express or implied, with respect to the information provided here.
