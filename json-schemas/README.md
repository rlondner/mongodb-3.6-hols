# Strict Document Validation

## Introduction

MongoDB 3.6 includes support for Strict Document Validation, based on the [JSON Schemas draft specification](http://json-schema.org/). This JSON Schemas support extends the capabilities of the [document validation](https://docs.mongodb.com/manual/core/document-validation/) feature introduced in MongoDB 3.2.

As of MongoDB 3.2, [Document Validation](https://docs.mongodb.com/manual/core/document-validation/) can be used to require that any documents inserted or updated follow a set of validation rules expressed using MongoDB query syntax. This allows for definition of required content. However, it has no mechanism to restrict users from adding documents containing fields beyond those specified in the validation rules, nor for specifying completely required content including what's embedded inside arrays.

## Labs

The following labs are currently available:

* [Getting Started with Schema Validation](./HOL-PART1.md)
* Part 2 (TBC)