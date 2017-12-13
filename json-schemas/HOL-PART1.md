# Part 1: Getting started with Schema Validation

1. Install MongoDB 3.6 as specified in [Pre-requisites section](./../README.md).
1. In the Terminal/Command Prompt window running `mongo`, enter the following command:

   ```javascript
       use demo
       db.adminCommand( {setFeatureCompatibilityVersion: "3.6"}) //necessary only if you use a pre-3.6 version of the Mongo Shell
       db.testCol.drop()
       db.createCollection( "testCol",
       {
           validator:
              {
                  $jsonSchema:
               {
                   bsonType: "object",
                   required: ["b", "c", "d"],
                   properties:
                   {
                       a: {},  // 'a' can be anything, even missing
                       b: {
                         bsonType: "string",
                         description: "'b' must be a string and is required"
                       },
                       c: {
                         bsonType: ["double", "decimal"],
                         description: "'c' must be a double or a decimal and is required"
                       },
                       d:
                       {
                         bsonType: ["int", "string"],
                         pattern: "\\d",
                         minimum: 0,
                         maximum: 100,
                         exclusiveMaximum: true,
                         description: "'d' is a string that matches pattern, or a number in [0, 100) and is required"
                       }
                   }
               }
           }
       })
   ```

   This creates a `testCol` collection requiring that any document inserted or updated in the `testCol` collection complies with the following constraints:

   * `b`, `c` and `d` are required fields
   * `b` must be a string and is required
   * `c` must be a double or a decimal and is required
   * `d` is a string that matches pattern, or a number in [0, 100) and is required

1. Enter the following commands to test schema validation:

   ```javascript
   db.testCol.insertOne({ b: "test", d: "99", c: 1 }); //works since b is a string, c is a double (in JavaScript) and d is a string that matches the digit pattern

   db.testCol.insertOne({ b: "test", d: 99, c: 1 }); //doesn't work, since 99 is considered a double in JavaScript (and d must be an Int32 or a string matching the `\d` pattern)

   db.testCol.insertOne({ b: "test", d: "test", c: 1 }); //doesn't work, since d doesn't match the pattern

   db.testCol.insertOne({ b: "test", d: "99.1.1", c: 1 }); //works because 99.1.1 is a valid \d pattern

   db.testCol.insertOne({ b: "test", d: NumberInt(99), c: 1 }); //works because d is a valid Int32

   db.testCol.insertOne({ b: "test", d: NumberInt(100), c: 1 }); //doesn't work because d is not strictly lower than 100

   db.testCol.insertOne({ b: "test", d: NumberInt(0), c: 1 }); //works because 0 is a valid d value

   db.testCol.insertOne({ b: "test", d: NumberInt(-1), c: 1 }); //doesn't work because d cannot be negative
   ```

1. Let's add an additional `e` field and constraint (e can only be equal to 43 or "mongodb") by running the following command:

   ```javascript
   db.testCol.drop();
   db.createCollection("testCol", {
     validator: {
       $jsonSchema: {
         bsonType: "object",
         required: ["b", "c", "d"],
         properties: {
           a: {}, // 'a' can be anything, even missing
           b: {
             bsonType: "string",
             description: "'b' must be a string and is required"
           },
           c: {
             bsonType: ["double", "decimal"],
             description: "'c' must be a double or a decimal and is required"
           },
           d: {
             bsonType: ["int", "double"],
             minimum: 0,
             maximum: 100,
             exclusiveMaximum: true,
             description:
               "'d' is an integer or double in [0, 100) and is required"
           },
           e: {
             enum: [null, 42, "mongodb"],
             description:
               "'e' can only be one of the given enum values or missing"
           }
         }
       }
     }
   });
   ```

   Run the following command to test the new JSON schema:

   ```javascript
   db.testCol.insertOne({ b: "test", d: 9, c: 1, e: 42 }); //works because 42 is a valid value for `e`

   db.testCol.insertOne({ b: "test", d: 9, c: 1, e: "mongodb" }); //works because "mongodb" is a valid value for `e`

   db.testCol.insertOne({ b: "test", d: 9, c: 1, e: 43 }); //doesn't work because 43 is an invalid value for `e`
   ```

   Note that we also switched `d` from being an int or a string (`bsonType: ["int", "string"]`) to being an int or a double (`bsonType: ["int", "double"]`) as it's not really realistic (nor is it a best practice) to have a numerical property stored as a string. We only had `bsonType: ["int", "string"]` to illustrate the use of the `pattern` keyword with regular expressions (since it applies to the `string` type).
