# GraphQL

## Introduction

- A query language for APIs
- Uses a type system that a user defines for their data
    - Both data schema and functions to execute upon query reception are both defined by users
- Once a GraphQL service is running it can be sent queries to validate and execute
    - A received query is first checked to make sure it references user defined types and fields
    - Then it is executed with the provided functions

## Queries and Mutations

- Example of a query:

    {
        hero {
            name
        }
    }
- Query fields can also refer to other data objects so instead of hitting multiple REST endpoints we can send one query
    - Great for relational data (think of the Star Wars R2-D2 example ({friends {name }}))
- Can pass arguments to the field
    - Example: 
        {
            hero(id: "1") {
                name
                height
            }
        }
    - The argument(id) allows us to grab just one element in our data that we need
    - In GraphQL every field and nested object can have arguments
        - We can even pass arguments into scalar fields to do data transformations just once on the server
        - Example (will convert height to feet on the fly):
            {
            hero(id: "1") {
                name
                height(unit: FOOT)
            }
        }
- We have the ability to alias fields to rename the result of a field to anything we want
    - Example:
        {
            luke: hero(episode: EMPIRE) {
                name
            }
            r2: hero(episode: JEDI) {
                name
            }
        }
    - The data coming back will now be accessible on luke and on r2