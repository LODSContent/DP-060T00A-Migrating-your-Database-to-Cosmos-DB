# Lab: Migrate Cassandra to Cosmos DB

## Estimated Time

90 minutes

## Scenario

In this lab, you'll migrate two datasets from Cassandra to Cosmos DB. You'll move the data in two ways. First, you'll export the data from Cassandra and use the CQLSH COPY command to import the database into Cosmos DB. Then, you'll migrate the data using Spark. You'll verify that the migration was successful by running an application that queries data held in the original Cassandra database, and then reconfiguring the application to connect to Cosmos DB. The results of running the reconfigured application should remain the same.

The scenario for this lab concerns an ecommerce system. Customers can place orders for goods. The customer and order details are recorded in a Cassandra database. You have an application that generates summaries, such as the list of orders for a customer, the orders for a specific product, and various aggregations, such as the number of orders placed, and so on.

The lab runs using the Azure Cloud shell and the Azure portal.

## Objectives

In this lab, you will:

* Export the schema
* Move data using CQLSH COPY
* Move data using Spark
* Verify the migration

## Notes

The full instructions for this lab are available at https://github.com/MicrosoftLearning/DP-060T00A-Migrating-your-Database-to-Cosmos-DB/blob/master/Labs/Lab%203%20-%20Migrate%20Cassandra%20Workloads%20to%20Cosmos%20DB.md

All of the files for this lab are located at https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB.

Students should download a copy of the lab files before performing the lab.

## Lab Exercises

* Export the Schema
* Move Data Using CQLSH COPY
* Move Data Using Spark
* Verify Migration

## Lab Review

In this lab, you migrated two datasets from Cassandra to Cosmos DB. You moved the data in two ways. First, you exported the data from Cassandra and used the CQLSH COPY command to import the database into Cosmos DB. Then, you migrated the data using Spark. You verified that migration was successful by running an application that queries data held in the original Cassandra database, and then reconfiguring the application to connect to Cosmos DB.
