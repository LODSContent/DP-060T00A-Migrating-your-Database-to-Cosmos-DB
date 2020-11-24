# Lab: Migrate MongoDB to Cosmos DB

## Estimated Time

90 minutes

## Scenario

In this lab, you'll take an existing MongoDB database and migrate it to Cosmos DB. You'll use the Azure Database Migration Service. You'll also see how to reconfigure existing applications that use the MongoDB database to connect to the Cosmos DB database instead.

The lab is based around an example system that captures temperature data from a series of IoT devices. The temperatures are logged in a MongoDB database, together with a timestamp. Each device has a unique ID. You will run a MongoDB application that simulates these devices and stores the data in the database. You will also use a second application that enables a user to query statistical information about each device. After migrating the database from MongoDB to Cosmos DB, you'll configure both applications to connect to Cosmos DB, and verify that they still function correctly.

The lab runs using the Azure Cloud shell and the Azure portal.

## Objectives

In this lab, you will:

* Create a migration project
* Define the source and target for your migration
* Perform the migration
* Verify the migration

## Notes

The complete instructions for this lab are available at https://github.com/MicrosoftLearning/DP-060T00A-Migrating-your-Database-to-Cosmos-DB/blob/master/Labs/Lab%202%20-%20Migrate%20MongoDB%20Workloads%20to%20Cosmos%20DB.md.

All of the files for this lab are located at https://github.com/MicrosoftLearning/DP-160T00A-Migrating-your-Database-to-Cosmos-DB.

Students should download a copy of the lab files before performing the lab.

## Lab Exercises

* Create a Migration Project
* Define Source and Target
* Perform the Migration
* Verify the Migration

## Lab Review

In this lab, you took an existing MongoDB database and migrated it to Cosmos DB. You used the Azure Database Migration Service. You also saw how to reconfigure existing applications that use the MongoDB database to connect to the Cosmos DB database instead.
