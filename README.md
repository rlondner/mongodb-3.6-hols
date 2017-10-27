# MongoDB 3.6 Hands-on Labs

This repository contains hands-on labs for the following new features in MongoDB 3.6:

* Change Streams
* [Strict Document Validation](./json-schemas) (aka JSON Schema support)
* Aggregation Framework improvements

## Pre-requisites

For all the hands-on labs available in this repository, you will need to take the following steps to install the latest version of MongoDB 3.6 on your machine:

Download the latest version of MongoDB 3.6 from the [Download Center](https://www.mongodb.com/download-center#community). At the time of this writing, this latest version is the Release Candidate 1, available in the Development Releases section.

1. Create a `data` folder in your MongoDB 3.6 root folder

1. From your MongoDB 3.6  root folder, start mongod with the following command:

        ./bin/mongod --dbpath ./data

1. In a separate Terminal/Command Prompt window from your MongoDB 3.6 root folder, start the Mongo shell by typing 

       ./bin/mongo

1. Type the following to switch to the `test` database

        use test
