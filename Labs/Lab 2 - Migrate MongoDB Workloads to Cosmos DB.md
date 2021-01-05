
# Lab 2: Migrate MongoDB Workloads to Cosmos DB
<!-- TOC -->

- [Lab 2: Migrate MongoDB Workloads to Cosmos DB](#lab-2-migrate-mongodb-workloads-to-cosmos-db)
    - [Exercise 1: Setup](#exercise-1-setup)
        - [Task 1: Create a Resource Group and Virtual Network](#task-1-create-a-resource-group-and-virtual-network)
        - [Task 2: Create a MongoDB Database Server](#task-2-create-a-mongodb-database-server)
        - [Task 3: Configure the MongoDB Database](#task-3-configure-the-mongodb-database)
    - [Exercise 2: Populate and Query the MongoDB Database](#exercise-2-populate-and-query-the-mongodb-database)
        - [Task 1: Build and Run an App to Populate the MongoDB Database](#task-1-build-and-run-an-app-to-populate-the-mongodb-database)
        - [Task 2: Build and Run Another App to Query the MongoDB Database](#task-2-build-and-run-another-app-to-query-the-mongodb-database)
    - [Exercise 3: Migrate the MongoDB Database to Cosmos DB](#exercise-3-migrate-the-mongodb-database-to-cosmos-db)
        - [Task 1: Create a Cosmos Account and Database](#task-1-create-a-cosmos-account-and-database)
        - [Task 2: Create the Database Migration Service](#task-2-create-the-database-migration-service)
        - [Task 3: Create and Run a New Migration Project](#task-3-create-and-run-a-new-migration-project)
        - [Task 4: Verify that Migration was Successful](#task-4-verify-that-migration-was-successful)
    - [Exercise 4: Reconfigure and Run Existing Applications to Use Cosmos DB](#exercise-4-reconfigure-and-run-existing-applications-to-use-cosmos-db)
    - [Exercise 5: Clean Up](#exercise-5-clean-up)

<!-- /TOC -->
In this lab, you'll take an existing MongoDB database and migrate it to Cosmos DB. You'll use the Azure Database Migration Service. You'll also see how to reconfigure existing applications that use the MongoDB database to connect to the Cosmos DB database instead.

The lab is based around an example system that captures temperature data from a series of IoT devices. The temperatures are logged in a MongoDB database, together with a timestamp. Each device has a unique ID. You will run a MongoDB application that simulates these devices, and stores the data in the database. You will also use a second application that enables a user to query statistical information about each device. After migrating the database from MongoDB to Cosmos DB, you'll configure both applications to connect to Cosmos DB, and verify that they still function correctly.

The lab runs using the Azure Cloud shell and the Azure portal.

## Exercise 1: Setup

In the first exercise, you'll create the MongoDB database for holding the data captured from temperature devices that your company manufactures.

### Task 1: Create a Resource Group and Virtual Network

1. In your Internet browser, navigate to https://portal.azure.com and sign in.
1. In the Azure portal, select **Resource groups**, and then select **+Add**.
1. On the **Create a resource group page**, enter the following details:

    | Property  | Value  |
    |---|---|
    | Subscription | *\<your-subscription\>* |
    | Resource Group | mongodbrg |
    | Region | Select your nearest location |

1. Select **Review + Create** and then select **Create**. Wait for the resource group to be created.
1. In the hamburger menu of the Azure portal, select **+ Create a resource**.
1. On the **New** page, in the **Search the Marketplace** box, type **Virtual Network**, and press Enter.
1. On the **Virtual Network** page, select **Create**.
1. On the **Create virtual network** page, enter the following details:

    | Property  | Value  |
    |---|---|
    | Subscription | *\<your-subscription\>* |
    | Resource Group | mongodbrg |
    | Name | databasevnet |
    | Region | Select the same location that you specified for the resource group |

1. Select **Next: IP Addresses**
1. On the **IP Addresses** page, under **IPv4 address space**, select the **10.0.0.0/16** space, and then to the right select the **Delete** button.
1. In the **IPv4 address space** list, enter **10.0.0.0/24**.
1. Select **+ Add subnet**. In the **Add subnet** pane, set the **Subnet name** to **default**, set the **Subnet address range** to **10.0.0.0/28**, and then select **Add**.
1. On the **IP Addresses** page, select **Next: Security**.
1. On the **Security** page, verify that **DDoS Protection Standard** is set to **Disable**, and **Firewall** is set to **Disable**. Select **Review * create**.
1. On the **Create virtual network** page, select **Create**. Wait for the virtual network to be created before continuing.

### Task 2: Create a MongoDB Database Server

1. In the hamburger menu of the Azure portal, select **+ Create a resource**.
1. In the **Search the Marketplace** box, type **Ubuntu**, and then press Enter.
1. On the **Marketplace** page, select **Ubuntu Server 18.04 LTS**. 
1. On the **Ubuntu Server 18.04 LTS** page, select **Create**.
1. On the **Create a virtual machine** page, enter the following details:

    | Property  | Value  |
    |---|---|
    | Subscription | *\<your-subscription\>* |
    | Resource Group | mongodbrg |
    | Virtual machine name | mongodbserver | 
    | Region | Select the same location that you specified for the resource group |
    | Availability options | No infrastructure redundancy required |
    | Image | Ubuntu Server 18.04 LTS - Gen1 |
    | Azure Spot instance | Unchecked |
    | Size | Standard A1_v2 |
    | Authentication type | Password |
    | Username | azureuser |
    | Password | Pa55w.rdPa55w.rd |
    | Confirm password | Pa55w.rdPa55w.rd |
    | Public inbound ports | Allow selected ports |
    | Select inbound ports | SSH (22) |

1. Select **Next: Disks \>**.
1. On the **Disks** page, leave the settings at their default, and then select **Next: Networking \>**.
1. On the **Networking** page, enter the following details:

    | Property  | Value  |
    |---|---|
    | Virtual network | databasevnet |
    | Subnet | default (10.0.0.0/28) |
    | Public IP | (new) mongodbserver-ip |
    | NIC network security group | Advanced |
    | Configure network security group | (new) mongodbserver-nsg |
    | Accelerated networking | Unchecked |
    | Load balancing | Unchecked |

1. Select **Review + create \>**.
1. On the validation page, select **Create**.
1. Wait for the virtual machine to be deployed before continuing
1. In hamburger menu of the Azure portal, select **All resources**.
1. On the **All resources** page, select **mongodbserver-nsg**.
1. On the **mongodbserver-nsg** page, under **Settings**, select **Inbound security rules**.
1. On the **mongodbserver-nsg - Inbound security rules** page, select **+ Add**.
1. In the **Add inbound security rule** pane, enter the following details:

    | Property  | Value  |
    |---|---|
    | Source | Any |
    | Source port ranges | * |
    | Destination | Any |
    | Destination port ranges | 27017 |
    | Protocol | Any |
    | Action | Allow |
    | Priority | 1030 |
    | Name | Mongodb-port |
    | Description | Port that clients use to connect to MongoDB |

1. Select **Add**.

### Task 3: Install MongoDB

1. In the hamburger menu the Azure portal, select **All resources**.
1. On the **All resources** page, select **mongodbserver-ip**.
1. On the **mongodbserver-ip** page, make a note of the **IP address**.
1. In the toolbar at the top of the Azure portal, select **Cloud Shell**.
1. If the **You have no storage mounted** message box appears, select **Create storage**.
1. When the Cloud Shell starts, in the drop-down list above the Cloud Shell window, select **Bash**.
1. In the Cloud Shell, enter the following command to connect to the mongodbserver virtual machine. Replace *\<ip address\>* with the value of the **mongodbserver-ip** IP address:

    ```bash
    ssh azureuser@<ip address>
    ```

1. At the prompt, type **yes** to continue connecting.
1. Enter the password **Pa55w.rdPa55w.rd**.
1. To import the MongoDB public GPG key, enter this command. The command should return **OK**:

    ```bash
    wget -qO - https://www.mongodb.org/static/pgp/server-4.0.asc | sudo apt-key add -
    ```

1. To create a list of packages, enter this command:

    ```bash
    echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
    ```

    The command should return text similar to `deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.0 multiverse`. This indicates that the package list has been successfully created.

1. To reload the package database, enter this command:

    ```bash
    sudo apt-get update
    ```

1. To install MongoDB, enter this command:

    ```bash
    sudo apt-get install -y mongodb-org
    ```

    The installation should proceed with messages about installing, preparing, and unpacking packages. It can take a few minutes for the installation to complete.

### Task 4: Configure the MongoDB database

By default, the Mongo DB instance is configured to run without authentication. In this task, you'll configure MongoDB to bind to the local network interface so that it can accept connections from other computers. You'll also enable authentication and create the necessary user account to perform migration. Finally, you'll add an account that a test application can use to query the database.

1. To open the MongoDB configuration file, run this command:

    ```bash
    sudo nano /etc/mongod.conf
    ```

1. In the file, locate the **bindIp** setting, and set it to **0.0.0.0**.
1. Add the following setting. If there is already a `security` section, replace it with this code. Otherwise add the code on two new lines:

    ```bash
    security:
        authorization: 'enabled'
    ```

1. To save the configuration file, press <kbd>Esc</kbd> and then press <kbd>CTRL + X</kbd>. Press <kbd>y</kbd> and then <kbd>Enter</kbd> to save the modified buffer.
1. To restart the MongoDB service and apply your changes, enter this command:

    ```bash
    sudo service mongod restart
    ```

1. To connect to the MongoDB service, enter this command:

    ```bash
    mongo
    ```

1. At the **>** prompt, to switch to the **admin** database, run this command:

    ```bash
    use admin;
    ```

1. To create a new user named **administrator**, run the following command. You can enter the command on one line or across multiple lines for better readability. The command is executed when the `mongo` program reaches the semicolon:

    ```bash
    db.createUser(
        {
            user: "administrator",
            pwd: "Pa55w.rd",
            roles: [
                { role: "userAdminAnyDatabase", db: "admin" },
                { role: "clusterMonitor", db:"admin" },
                "readWriteAnyDatabase"
            ]
        }
    );
    ```

1. To exit the `mongo` program, enter this command;

    ```bash
    exit;
    ```

1. To connect to MongoDB with the new administrator's account, run this command:

    ```bash
    mongo -u "administrator" -p "Pa55w.rd"
    ```

1. To switch to the **DeviceData** database, execute this command:

    ```bash
    use DeviceData;    
    ```

1. To create a user named **deviceadmin**, which the app will use to connect to the database, run this command:

    ```bash
    db.createUser(
        {
            user: "deviceadmin",
            pwd: "Pa55w.rd",
            roles: [ { role: "readWrite", db: "DeviceData" } ]
        }
    );
    ```

1. To exit the `mongo` program, enter this command;

    ```bash
    exit;
    ```

1. Run the following command restart the mongodb service. Verify that the service restarts without any error messages:

    ```bash
    sudo service mongod restart
    ```

1. Run the following command to verify that you can now log in to mongodb as the deviceadmin user:

    ```bash
    mongo -u "deviceadmin" -p "Pa55w.rd" --authenticationDatabase DeviceData
    ```

1. At the **>** prompt, run the following command to quit the mongo shell:

    ```bash
    exit;
    ```

1. At the bash prompt, run the following command to disconnect from the MongoDB server and return to the Cloud Shell:

    ```bash
    exit
    ```

## Exercise 2: Populate and Query the MongoDB Database

You have now created a MongoDB server and database. The next step is to demonstrate the sample applications that can populate and query the data in this database.

### Task 1: Build and Run an App to Populate the MongoDB Database

1. In the Azure Cloud Shell, run the following command to download the sample code for this workshop:

    ```bash
    git clone https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB migration-workshop-apps
    ```

1. Move to the **migration-workshop-apps/MongoDeviceDataCapture/MongoDeviceCapture** folder:

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/MongoDeviceDataCapture
    ```

1. Use the **Code** editor to examine the **TemperatureDevice.cs** file:

    ```bash
    code TemperatureDevice.cs
    ```

    The code in this file contains a class named **TemperatureDevice** that simulates a temperature device capturing data and saving it in a MongoDB database. It uses the MongoDB library for the .NET Framework. The  **TemperatureDevice** constructor connects to the database using settings stored in the application configuration file. The **RecordTemperatures** method generates a reading and writes it to the database.

1. To close the code editor press <kbd>CTRL + Q</kbd>, and then open the **ThermometerReading.cs** file:

   ```bash
   code ThermometerReading.cs
   ```

    This file shows the structure of the documents that the application stores in the database. Each document contains the following fields:

    - An object ID. The is the "_id" field generated by MongoDB to uniquely identify each document.
    - A device ID. Each device has a number with the prefix "Device".
    - The temperature recorded by the device.
    - The date and time when the temperature was recorded.
  
1. To close the code editor press <kbd>CTRL + Q</kbd>, and then open the **App.config** file:

    ```bash
    code App.config
    ```

    This file contains the settings for connecting to the MongoDB database. Set the value for the **Address** key to the IP address of the MongoDB server that you recorded earlier.

1. To save the file, press <kbd>CTRL + S</kbd>, and the to close the code editor press <kbd>CTRL + Q</kbd>
1. To open the project file, enter this command:

    ```bash
    code MongoDeviceDataCapture.csproj
    ```

1. Check that the `<TargetFramework>` tags contain the value `netcoreapp3.1`. Replace this value if necessary.
1. To save the file, press <kbd>CTRL + S</kbd>, and the to close the code editor press <kbd>CTRL + Q</kbd>
1. Run the following command to rebuild the application:

    ```bash
    dotnet build
    ```

1. Run the application:

    ```bash
    dotnet run
    ```

    The application simulates 100 devices running simultaneously. Allow the application to run for a couple of minutes, and then press Enter to stop it.

### Task 2: Build and run another app to query the MongoDB database

1. Move to the **migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery** folder:

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery
    ```

    This folder contains another application that you can use to analyze the data captured by each device.

1. Use the **Code** editor to examine the **Program.cs** file:

    ```bash
    code Program.cs
    ```

    The application connects to the database (using the **ConnectToDatabase** method at the bottom of the file) and then prompts the user for a device number. The application uses the MongoDB library for the .NET Framework to create and run an aggregate pipeline that calculates the following statistics for the specified device:

    - The number of readings recorded.
    - The average temperature recorded.
    - The lowest reading.
    - The highest reading.
    - The latest reading.

1. To close the code editor press <kbd>CTRL + Q</kbd>, and then open the **App.config** file:

    ```bash
    code App.config
    ```

    As before, set the value for the **Address** key to the IP address of the MongoDB server that you recorded earlier.

1. To save the file, press <kbd>CTRL + S</kbd>, and the to close the code editor press <kbd>CTRL + Q</kbd>
1. To open the project file, enter this command:

    ```bash
    code DeviceDataQuery.csproj
    ```

1. Check that the `<TargetFramework>` tags contain the value `netcoreapp3.1`. Replace this value if necessary.
1. To save the file, press <kbd>CTRL + S</kbd>, and the to close the code editor press <kbd>CTRL + Q</kbd>

1. Build and run the application:

    ```bash
    dotnet build
    dotnet run
    ```

1. At the **Enter Device Number** prompt, enter a value between 0 and 99. The application will query the database, calculate the statistics, and display the results. Press <kbd>Q + Enter</kbd> to quit the application.

## Exercise 3: Migrate the MongoDB Database to Cosmos DB

The next step is to take the MongoDB database and transfer it to Cosmos DB.

### Task 1: Create a Cosmos Account and Database

1. Return to the Azure portal.
1. In the hamburger menu, select **+ Create a resource**.
1. On the **New** page, in the **Search the Marketplace** box, type **Azure Cosmos DB**, end then press Enter.
1. On the **Azure Cosmos DB** page, select **Create**.
1. On the **Create Azure Cosmos DB Account** page, enter the following settings:

    | Property  | Value  |
    |---|---|
    | Subscription | Select your subscription |
    | Resource group | mongodbrg |
    | Account Name | mongodb*nnn*, where *nnn* is a random number selected by you |
    | API | Azure Cosmos DB for MongoDB API |
    | Notebooks | Off |
    | Location | Specify the same location that you used for the MongoDB server and virtual network |
    | Capacity mode | Provisioned throughput |
    | Apply Free Tier Discount | Apply |
    | Account Type | Non-Production |
    | Version | 3.6 |
    | Geo-Redundancy | Disable |
    | Multi-region Writes | Disable |
    | Availability Zones | Disable |

1. Select **Review + create**
1. On the validation page, select **Create**, and wait for the Cosmos DB account to be deployed.
1. In the hamburger menu of the Azure portal, select **All resources**, and then select your new Cosmos DB account (**mongodb*nnn***).
1. On the **mongodb*nnn*** page, select **Data Explorer**.
1. In the **Data Explorer** pane, select **New Collection**.
1. In the **Add Collection** pane, specify the following settings:

    | Property  | Value  |
    |---|---|
    | Database id | Select **Create new**, and then type **DeviceData** |
    | Provision database throughput | Checked |
    | Throughput | 1000 |
    | Collection id | Temperatures |
    | Storage capacity | Unlimited |
    | Shard key | deviceID |
    | My shard key is larger than 100 bytes | unchecked |
    | Create a Wildcard Index on all fields | unchecked |
    | Analytical store | Off |

1. Select **OK**.

### Task 2: Create the Database Migration Service

1. In the hamburger menu of the Azure portal, select **All services**.
1. In the **All services** search box, type **Subscriptions**, and then press Enter.
1. On the **Subscriptions** page, select your subscription.
1. On your subscription page, under **Settings**, select **Resource providers**.
1. In the **Filter by name** box, type **DataMigration**, and then select **Microsoft.DataMigration**.
1. If the **Status** is not **Registered**, select **Register**, and wait for the **Status** to change to **Registered**. It might be necessary to select **Refresh** to see the status change.
1. In the hamburger menu of the Azure portal, select **+ Create a resource**.
1. On the **New** page, in the **Search the Marketplace** box, type **Azure Database Migration Service**, and then press Enter.
1. On the **Azure Database Migration Service** page, select **Create**.
1. On the **Create Migration Service** page, enter the following settings:

    | Property  | Value  |
    |---|---|
    | Subscription | Select your subscription |
    | Resource group | mongodbrg |
    | Service Name | MongoDBMigration |
    | Location | Select the same location that you used previously |
    | Service mode | Azure |
    | Pricing Tier | Standard: 1 vCores |

1. Select **Next: Networking**.
1. On the **Networking** page, check the **databasevnet/default** checkbox, and then select **Review + create**
1. Select **Create**, and wait for the service to be deployed before continuing. This operation will take a few minutes.

### Task 3: Create and Run a New Migration Project

1. In the hamburger menu of the Azure portal, select **Resource groups**.
1. In the **Resource groups** window, select **mongodbrg**.
1. In the **mongodbrg** window, select **MongoDBMigration**.
1. On the **MongoDBMigration** page, select **+ New Migration Project**.
1. On the **New migration project** page, enter the following settings:

    | Property  | Value  |
    |---|---|
    | Project name | MigrateTemperatureData |
    | Source server type | MongoDB |
    | Target server type | Cosmos DB (MongoDB API) |
    | Choose type of activity | Offline data migration |

1. Select **Create and run activity**.
1. When the **Migration Wizard** starts, on the **Source details** page, enter the following details:

    | Property  | Value  |
    |---|---|
    | Mode | Standard mode |
    | Source server name | Specify the value of the **mongodbserver-ip** IP address that you recorded earlier |
    | Server port | 27017 |
    | User Name | administrator |
    | Password | Pa55w.rd |
    | Require SSL | unchecked |

1. Select **Next: Select target**.
1. On the **Select target** page, enter the following details:

    | Property  | Value  |
    |---|---|
    | Mode | Select Cosmos DB target |
    | Subscription | Select your subscription |
    | Select Cosmos DB name | mongodb*nnn* |
    | Connection string | Accept the connection string generated for your Cosmos DB account |

1. Select **Next: Database setting**.
1. On the **Database setting** page, enter the following details:

    | Property  | Value  |
    |---|---|
    | Source Database | DeviceData |
    | Target Database | DeviceData |
    | Throughput (RU/s) | 1000 |
    | Clean up collections | Unchecked |

1. Select **Next: Collection setting**.
1. On the **Collection setting** page, select the dropdown arrow by the DeviceData database, enter the following details:

    | Property  | Value  |
    |---|---|
    | Name | Temperatures |
    | Target Collection | Temperatures |
    | Throughput (RU/s) | 1000 |
    | Shard Key | deviceID |
    | Unique | Leave blank |

1. Select **Next: Migration summary**
1. On the **Migration summary** page, in the **Activity name** field, enter **mongodb-migration**, and then select **Start migration**.
1. On the **mongodb-migration** page, select **Refresh** every 30 seconds, until the migration has completed. Note the number of documents processed.

### Task 4: Verify that Migration was Successful

1. In the hamburger menu of the Azure portal, select **All Resources**.
1. On the **All resources** page, select **mongodb*nnn***.
1. On the **mongodb*nnn** page, select **Data Explorer**.
1. In the **Data Explorer** pane, expand the **DeviceData** database, expand the **Temperatures** collection, and then select **Documents**.
1. In the **Documents** pane, scroll through the list of documents. You should see a document id (**_id**) and the shard key (**/deviceID**) for each document.
1. Select any document. You should see the details of the document displayed. A typical document looks like this:

    ```JSON
    {
	    "_id" : ObjectId("5ce8104bf56e8a04a2d0929a"),
	    "deviceID" : "Device 83",
	    "temperature" : 19.65268837271849,
	    "time" : 636943091952553500
    }
    ```

1. In the toolbar in the **Document Explorer** pane, select **New Shell**.
1. In the **Shell 1** pane, at the **\>** prompt, enter the following command, and then press Enter:

    ```mongosh
    db.Temperatures.count()
    ```

    This command displays the number of documents in the Temperatures collection. It should match the number reported by the Migration Wizard .

1. Enter the following command, and then press Enter:

    ```mongosh
    db.Temperatures.find({deviceID: "Device 99"})
    ```

    This command fetches and displays the documents for Device 99.

## Exercise 4: Reconfigure and Run Existing Applications to Use Cosmos DB

The final step is to reconfigure your existing MongoDB applications to connect to Cosmos DB, and verify that they operate as before. This process requires you to modify the way in which your applications connect to the database, but the logic of your applications should remain unchanged.

1. In the **mongodb*nnn*** pane, under **Settings**, select **Connection String**.
1. On the **mongodb*nnn* Connection String** page, make a note of the following settings:

    - Host
    - Username
    - Primary Password
  
1. Return to the Cloud Shell window (reconnect if the session has timed out), and move to the **migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery** folder:

    ```bash
    cd ~/migration-workshop-apps/MongoDeviceDataCapture/DeviceDataQuery
    ```

1. Open the App.config file in the Code editor:

    ```bash
    code App.config
    ```

1. In the **Settings for MongoDB** section of the file, comment out the existing settings.
1. Uncomment the settings in the **Settings for Cosmos DB Mongo API** section, and set the values for these settings as follows:

    | Setting  | Value  |
    |---|---|
    | Address | The **Host** from the **mongodb*nnn* Connection String** page |
    | Username | The **Username** from the **mongodb*nnn* Connection String** page |
    | Password | The **Primary Password** from the **mongodb*nnn* Connection String** page |

    The completed file should look similar to this:

    ```XML
    <?xml version="1.0" encoding="utf-8"?>
    <configuration>
        <appSettings>
            <add key="Database" value="DeviceData" />
            <add key="Collection" value="Temperatures" />

            <!-- Settings for MongoDB -->
            <!--add key="Address" value="nn.nn.nn.nn" />
            <add key="Port" value="27017" />
            <add key="Username" value="deviceadmin" />
            <add key="Password" value="Pa55w.rd" /-->
            <!-- End of settings for MongoDB -->

            <!-- Settings for CosmosDB Mongo API -->
            <add key="Address" value="mongodbnnn.documents.azure.com"/>
            <add key="Port" value="10255"/>
            <add key="Username" value="mongodbnnn"/>
            <add key="Password" value="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=="/>
            <!-- End of settings for CosmosDB Mongo API -->
        </appSettings>
    </configuration>
    ```

1. Save the file, and then close the Code editor.

1. Open the Program.cs file using the Code editor:

    ```bash
    code Program.cs
    ```

1. Scroll down to the **ConnectToDatabase** method.
1. Comment out the line that sets the credentials for connecting to MongoDB, and uncomment the statements that specify the credentials for connecting to Cosmos DB. The code should look like this:

    ```C#
    // Connect to the MongoDB database
    MongoClient client = new MongoClient(new MongoClientSettings
    {
        Server = new MongoServerAddress(address, port),
        ServerSelectionTimeout = TimeSpan.FromSeconds(10),

        //
        // Credential settings for MongoDB
        //

        // Credential = MongoCredential.CreateCredential(database, azureLogin.UserName, azureLogin.SecurePassword),

        //
        // Credential settings for CosmosDB Mongo API
        //

        UseSsl = true,
        SslSettings = new SslSettings
        {
            EnabledSslProtocols = SslProtocols.Tls12
        },
        Credential = new MongoCredential("SCRAM-SHA-1", new MongoInternalIdentity(database, azureLogin.UserName), new PasswordEvidence(azureLogin.SecurePassword))

        // End of Mongo API settings 
    });

    ```

    These changes are necessary because the original MongoDB database was not using an SSL connection. Cosmos DB always uses SSL.

1. Save the file, and then close the Code editor.
1. Rebuild and run the application:

    ```bash
    dotnet build
    dotnet run
    ```

1. At the **Enter Device Number** prompt, enter a device number between 0 and 99. The application should run exactly as before, except this time it is using the data held in the Cosmos DB database.
1. Test the application with other device numbers. Enter **Q** to finish.

You have successfully migrated a MongoDB database to Cosmos DB, and reconfigured an existing MongoDB application to connect to the Cosmos DB database.

## Exercise 5: Clean Up

1. Return to the Azure portal.
1. In the hamburger menu, select **Resource groups**.
1. In the **Resource groups** window, select **mongodbrg**.
1. Select **Delete resource group**.
1. On the **Are you sure you want to delete "mongodbrg"** page, in the **Type the resource group name** box, enter **mongodbrg**, and then select **Delete**.

---
© 2020 Microsoft Corporation. All rights reserved.

The text in this document is available under the [Creative Commons Attribution 3.0 License](https://creativecommons.org/licenses/by/3.0/legalcode), additional terms may apply. All other content contained in this document (including, without limitation, trademarks, logos, images, etc.) are **not** included within the Creative Commons license grant. This document does not provide you with any legal rights to any intellectual property in any Microsoft product. You may copy and use this document for your internal, reference purposes.

This document is provided "as-is." Information and views expressed in this document, including URL and other Internet Web site references, may change without notice. You bear the risk of using it. Some examples are for illustration only and are fictitious. No real association is intended or inferred. Microsoft makes no warranties, express or implied, with respect to the information provided here.
