# Part 3: Array Validation

With our newly learned knowledge of validation for sub-documents, let's go a step further and look at how to validate another feature of MongoDB's JSON Schema Validation support: [arrays](https://tools.ietf.org/html/draft-fge-json-schema-validation-00#page-9).

We can use the `array` type (or bsonType), and the associated `items` property to define what our array looks like. `items` **must** either be an object or an array. Objects must be valid JSON Schema definitions and arrays **must** be objects with valid JSON schemas.

Our use case here will be the modeling of a cooking recipe. Typically, a recipe will have a list of ingredients. In a `recipes` collection then, our documents would have, among other things, an array of ingredients. Let's assume that for each ingredient listed we need a *quantity*, *measurement*, *ingredient*, and an optional *format*.

1. Let's define a *basic* recipe skeleton:

    ```javascript
        db.recipes.drop()
        db.createCollection ( "recipes",
        {
            validator:
                        {
            $jsonSchema:
        {
            bsonType: "object",
            required: ["name", "ingredients"],
            properties:
            {
                _id: {},
                name: {
                    bsonType: "string",
                    description: "'name' must be a string and is required"
                    },
                ingredients: {
                    bsonType: ["array"],
                    minItems: 1, // each recipe must have at least one ingredient
                    items: {
                        bsonType: ["object"],
                        required: ["quantity", "measurement", "ingredient_name"],
                        additionalProperties: false,
                        description: "'ingredients' must contain the stated fields",
                        properties: {
                            quantity: {
                            bsonType: ["double", "decimal"],
                            description: "'quantity' is required and is of double or decimal type"
                                },
                        measurement: {
                            enum: ["tsp", "Tbsp", "cup", "ounce", "pound",  "each"],
                            description: "'measurement' is required and can only be one of the given enum values"
                                },
                        ingredient_name: {
                            bsonType: "string",
                            description: "'ingredient_name' is required and is a string"
                                },
                        format: {
                            bsonType: "string",
                            description: "'format' is an optional field of type string"
                                }
                        }
                    }
                }

                    }
                }
            }
        }
        )
    ```

2. In our recipe validation rule above we have required each recipe document to have a name and at least one ingredient. We've further defined what an **ingredient** looks like. Let's test our new rules:

    ```javascript
    db.recipes.insertOne({name: "Chocolate Ganache", ingredients: [
    {quantity: 7, measurement: "ounce", ingredient_name: "bittersweet chocolate", format: "chopped"},
    {quantity: 2, measurement: "cup", ingredient_name: "heavy cream"}
    ]})  // works
    db.recipes.insertOne({name: "Cheese Omelet", ingredients: [
    {quantity: .25, measurement: "cup", ingredient_name: "cheddar cheese", format: "shredded"},
    {quantity: 2, ingredient_name: "eggs"}
    ]})  // doesn't work because eggs is missing a measurement
    db.recipes.insertOne({name: "Milk",
    ingredients: {quantity: 1, measurement: "cup", ingredient_name: "Skim Milk", format: "chilled"}
    })  // doesn't work because ingredients isn't an array

    ```

3. Next, let's look at the `uniqueItems` keyword. This enforces items in an array to be, well, unique. Imagine if you work for a coloring crayon company which sells boxes of crayons. You want to make sure that each box of crayons includes different colors. How exciting would it be for young Martina to only get a box of *gray* crayons?

    ```javascript
        db.crayonBoxes.drop()
        db.createCollection ( "crayonBoxes",
        {
            validator:
                        {
            $jsonSchema:
        {
            bsonType: "object",
            required: ["name", "crayons"],
            properties:
            {
                _id: {},
                name: {
                    bsonType: ["string"],
                    description: "'name' is a required string"
                },
                crayons: {
                    bsonType: ["object"],
                    required: ["size", "colors"],
                    additionalProperties: false,
                    properties: {
                        size: {
                              enum: ["small", "medium", "large"],
                              description: "'size' is required and can only be one of the given enum values"
                            },
                        colors: {
                            bsonType: ["array"],
                            minItems: 1, // each box of crayons must have at least one color
                            uniqueItems: true,
                            items: {
                                bsonType: "string",
                                description: "'colors' is a required array of unique strings"
                                 }
                            }
                        }
                    }
                    }
                }
            }
        }
        )
    ```

4. We have set our validation rule here so that each box of crayons cannot contain identical crayons (i.e. of the same size and color).

    ```javascript
        db.crayons.insertOne({name: "Reds",
        crayons: {size: "small", colors: ["red", "maroon"]}}) // works

        db.crayons.insertOne({name: "Reds",
        crayons: {size: "small", colors: ["red", "maroon", "scarlet", "brick red", "red"]}})  // doesn't work since "red" is repeated
    ```

#### Note:
`uniqueItems` should be used for scalar arrays and **not** for
item arrays.