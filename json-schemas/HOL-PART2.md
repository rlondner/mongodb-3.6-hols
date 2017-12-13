# Part 2: Digging deeper into Schema Validation: sub-documents and additional fields restrictions

Now that you have made yourself more familiar with MongoDB’s JSON Schema support, let’s explore more advanced schema customizations.

1. Let's add another `f` property to test schema validation on sub-documents:

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
               "'d' is an integer or a double in [0, 100) and is required"
           },
           e: {
             enum: [null, 42, "mongodb"],
             description:
               "'e' can only be one of the given enum values or missing"
           },
           f: {
             bsonType: "object",
             properties: {
               a: { bsonType: "int" }
             },
             description: "'f' is an object with int field 'a'"
           }
         }
       }
     }
   });
   ```

   In the code above, we added a rule for an additional `f` property at the document root level, with the additional constraint that, if present, it should be of type ‘object’ (i.e. a sub-document, not a single property).
   Let's test the new schema validation rule to make sure it complies with our intent:

   ```javascript
   db.testCol.insertOne({ b: "test", d: 9, c: 1, e: 42, f: NumberInt(1) });
   //doesn't work because f should be a sub-document

   db.testCol.insertOne({ b: "test", d: 9, c: 1, e: 42, f: { a: "1" } });
   //doesn't work because a is not an integer

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) }
   }); //works

   db.testCol.insertOne({ b: "test", d: 9, c: 1, e: 42, f: { b: "1" } });
   //works because f is a sub-document but f.a is not required
   ```

1. Let's modify the schema a bit further to make the `f.a` element required:

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
               "'d' is an integer or a double in [0, 100) and is required"
           },
           e: {
             enum: [null, 42, "mongodb"],
             description:
               "'e' can only be one of the given enum values or missing"
           },
           f: {
             bsonType: "object",
             required: ["a"],
             properties: {
               a: { bsonType: "int" }
             },
             description: "'f' is an object with int field 'a'"
           }
         }
       }
     }
   });
   ```

   Let's test again:

   ```javascript
   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) }
   }); //still works

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { b: "1" }
   }); // no longer works because f.a is now required

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1), b: "1" }
   }); //works because f.a is present but there's no restriction on additional properties
   ```

1. As you may have noticed, we can add additional fields to the `f` sub-document, but we’d like to make sure that only the `a` field be added. In order to do so, let’s restrict the `f` sub-schema to only the `f.a` property by using the `additionalProperties` attribute:

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
               "'d' is a integer or a double in [0, 100) and is required"
           },
           e: {
             enum: [null, 42, "mongodb"],
             description:
               "'e' can only be one of the given enum values or missing"
           },
           f: {
             bsonType: "object",
             required: ["a"],
             properties: {
               a: { bsonType: "int" }
             },
             additionalProperties: false,
             description: "'f' is an object with only an 'a' int field"
           }
         }
       }
     }
   });
   ```

   By setting `additionalProperties` to `false`, we were able to enforce that the `f` sub-document only contain the properties specified in its `properties` section. By specifying these properties as required (using the `required` keyword), we make sure that these properties and **only they** are always present in new or updated documents (provided the `validationLevel` is set to `strict`, which is the default value). Now the following command no longer works because `f.a` is the only allowed property in the `f` sub-document:

   ```javascript
   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1), b: "1" }
   });
   ```

   Note that the `additionalProperties` keyword can also be used at the root level but the trick is to add the `_id` field in the `properties` section since it's a mandatory field MongoDB will automatically insert if it's not explicitly specified. For instance, you would have to specify the following schema if you wanted to restrict the addition of unspecified fields at the document root level:

   ```javascript
   db.testCol.drop();
   db.createCollection("testCol", {
     validator: {
       $jsonSchema: {
         bsonType: "object",
         required: ["b", "c", "d"],
         additionalProperties: false,
         properties: {
           _id: {},
           b: {
             bsonType: "string",
             description: "'b' must be a string and is required"
           },
           c: {
             bsonType: ["double", "decimal"],
             description: "'c' must be a double or a decimal and is required"
           },
           d: {
             bsonType: ["int", "string"],
             pattern: "\\d",
             minimum: 0,
             maximum: 100,
             exclusiveMaximum: true,
             description:
               "'d' is a string that matches pattern, or a number in [0, 100) and is required"
           },
           e: {}
         }
       }
     }
   });
   ```

   With the schema above, the following insert works:

         db.testCol.insertOne({b:"test",d:"9",c:1})

1. As an exercise, let’s add a few more fields (`g`, `h` and `i`) and constraints:

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
               "'d' is an interger or a double in [0, 100) and is required"
           },
           e: {
             enum: [null, 42, "mongodb"],
             description:
               "'e' can only be one of the given enum values or missing"
           },
           f: {
             bsonType: "object",
             required: ["a"],
             properties: {
               a: { bsonType: "int" }
             },
             additionalProperties: false,
             description: "'f' is an object with int field 'a'"
           },
           g: {
             properties: {
               a: { bsonType: "int" }
             },
             description: "like 'f', but no constraints if 'g' is not an object"
           },
           h: {
             bsonType: "object",
             required: ["a", "b", "c"],
             maxProperties: 3,
             description: "'h' is an object with exactly the required fields"
           },
           i: {
             maxItems: 2,
             description:
               "'i' has at most 2 items if an array; unconstrained otherwise"
           },
           j: {
             not: {
               enum: [null]
             },
             description: "'j' cannot be null"
           }
         }
       }
     }
   });
   ```

   Now, it’s up to you to figure out which of the following queries work. Good luck!

   ```javascript
   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     g: 1
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     g: { a: 1 }
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     g: { a: NumberInt(1) }
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     g: { b: 1 }
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     h: { a: 1, b: 1 }
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     h: { a: 1, b: 1, c: 1 }
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     h: { a: 1, b: 1, c: 1, d: 1 }
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     i: 1
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     i: { a: 1, b: 2, c: 3 }
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     i: [1, 2, 3]
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     i: [1, 2]
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     j: null
   });

   db.testCol.insertOne({
     b: "test",
     d: 9,
     c: 1,
     e: 42,
     f: { a: NumberInt(1) },
     j: ""
   });
   ```
