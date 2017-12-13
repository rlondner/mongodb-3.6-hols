# Part 4: Schema Dependencies

Another interesting feature offered in [JSON Schema Validation](https://tools.ietf.org/html/draft-fge-json-schema-validation-00#section-5.4.5)
is the utilization of the `dependencies` keyword to allow the schema to be
defined based on specific properties. This can be of great benefit to
move application logic natively into the database. These dependencies
come in two forms:

* **Property dependencies** state that if a specified property is present, other properties must be present as well.
* **Schema dependencies** define a _change_ in the schema when a given property is present.

## Property Dependencies

1. For the case of property dependencies, let's suppose our schema represents students and whether they have graduated from a program or not. If they have graduated, we want to be able to stay in touch with them and need their mailing address.

   ```javascript
   db.students.drop();
   db.createCollection("students", {
     validator: {
       $jsonSchema: {
         bsonType: "object",
         required: ["name"],
         properties: {
           _id: {},
           name: {
             bsonType: ["string"],
             description: "'name' is a required string"
           },
           graduated: {
             bsonType: ["bool"],
             description: "'graduated' is an optional boolean value"
           },
           mailing_address: {
             bsonType: ["string"],
             description: "'mailing_address' is an optional string value"
           }
         },
         dependencies: {
           graduated: ["mailing_address"]
         }
       }
     }
   });
   ```

1. We can test our schema validation rules:

   ```javascript
   db.students.insertOne({
     name: "Alena Weber",
     graduated: true,
     mailing_address: "123 Main Street"
   }); // works

   db.students.insertOne({
     name: "Jamal Aattou",
     graduated: true
   }); // doesn't work, no mailing_address

   db.students.insertOne({
       name: "Shannon Bradshaw"
    }); // works

   db.students.insertOne({
     name: "Andrei Popov",
     mailing_address: "456 Oak Street"
   }); // works, having an address without graduation is fine, dependencies are not bidirectional
   ```

    Note: If we set `graduated: false`, `mailing_address` would still be required from a schema validation standpoint and it would, therefore, be up to the application to not set a `graduated` attribute unless that condition is indeed true.

1. To make the dependencies bidirectional, we have to define them explicitly:

   ```javascript
   db.students.drop();
   db.createCollection("students", {
     validator: {
       $jsonSchema: {
         bsonType: "object",
         required: ["name"],
         properties: {
           _id: {},
           name: {
             bsonType: ["string"],
             description: "'name' is a required string"
           },
           graduated: {
             bsonType: ["bool"],
             description: "'graduated' is an optional boolean value"
           },
           mailing_address: {
             bsonType: ["string"],
             description: "'mailing_address' is an optional string value"
           }
         },
         dependencies: {
           graduated: ["mailing_address"],
           mailing_address: ["graduated"]
         }
       }
     }
   });
   ```

1. Neither of the following would pass validation:

   ```javascript
   db.students.insertOne({
     name: "Chris T. Barker",
     graduated: true
   }); // doesn't work, it has 'graduated', but is missing 'mailing_address'

   db.students.insertOne({
     name: "Courtney DeSaja",
     mailing_address: "789 Broadway"
   }); // doesn't work, it has 'mailing_address', but is missing 'graduated'
   ```

## Schema Dependencies

1. Schema dependencies work like property dependencies, but instead of just specifying other required properties, they can extend the schema to have other properties. Using the _schema dependency_ technique, the example from **1.** above can be rewritten as follows:

   ```javascript
   db.students.drop();
   db.createCollection("students", {
     validator: {
       $jsonSchema: {
         bsonType: "object",
         required: ["name"],
         properties: {
           _id: {},
           name: {
             bsonType: ["string"],
             description: "'name' is a required string"
           },
           graduated: {
             bsonType: ["bool"],
             description: "'graduated' is an optional boolean value"
           }
         },
         dependencies: {
           graduated: {
             required: ["mailing_address"],
             properties: {
               mailing_address: {
                 bsonType: ["string"]
               }
             }
           }
         }
       }
     }
   });
   ```

   In the schema above, note that the `mailing_address` property is no longer declared in the top-level `properties` section. Instead, it's declared in the `properties` section of the `graduated` dependency.

1. To test this new schema, run the following commands to test the updated JSON schema:

   ```javascript
   db.students.insertOne({
     name: "Alena Weber",
     graduated: true,
     mailing_address: "123 Main Street"
   }); // works

   db.students.insertOne({
     name: "Jamal Aattou",
     graduated: true
   }); // doesn't work, no mailing_address

   db.students.insertOne({
     name: "Chris T. Barker",
     mailing_address: "987 Pine St NE"
   }); // works
   ```
