# GraphQL

## Introduction

- A query language for APIs
- Uses a type system that a user defines for their data
    - Both data schema and functions to execute upon query reception are both defined by users
- Once a GraphQL service is running it can be sent queries to validate and execute
    - A received query is first checked to make sure it references user defined types and fields
    - Then it is executed with the provided functions

## Accessing Data

### Queries

- Example of a query:

    ```graphQL
    query getMeAHero {
        hero {
            name
        }
    }
    ```
- Query fields can also refer to other data objects so instead of hitting multiple REST endpoints we can send one query
    - Great for relational data (think of the Star Wars R2-D2 example ({friends {name }}))
- Can pass arguments to the field
    - Example:
        ```graphQL
        {
            hero(id: "1") {
                name
                height
            }
        }
        ```
    - The argument(id) allows us to grab just one element in our data that we need
    - In GraphQL every field and nested object can have arguments
        - We can even pass arguments into scalar fields to do data transformations just once on the server
        - Example (will convert height to feet on the fly):
            ```graphQL
            {
                hero(id: "1") {
                    name
                    height(unit: FOOT)
            }
            ```
- We have the ability to alias fields to rename the result of a field to anything we want
    - Example:
        ```graphQL
        {
            luke: hero(episode: EMPIRE) {
                name
            }
            r2: hero(episode: JEDI) {
                name
            }
        }
        ```
    - The data coming back will now be accessible on luke and on r2

### Fragments

- Fragments allow us to construct sets of fields (small queries) that can be reused in other (larger) queries
    - Example: 
    ```graphQL
    {
        leftComparison: hero(episode: EMPIRE) {
            ...comparisonFields
        }
        rightComparison: hero(episode: JEDI) {
            ...comparisonFields
        }
    }

    fragment comparisonFields on Character {
        name
        appearsIn
        friends {
            name
        }
    }
    ```

### Operation Types & Names

- Three types of operations ```query, mutation, subscription``` that describe what we're intending to do
- The operation name is a meaningful and explicit name for our operation
    - Useful for debugging
    - Just like a function name

### Variables

- Denoted with the $
- When we start working with variables, we need to do three things:
    - Replace the static value in the query with $variableName
    - Declare $variableName as one of the variables accepted by the query
    - Pass variableName: value in the separate, transport-specific (usually JSON) variables dictionary
- Example of using a variable:
    ```graphQL
    query HeroNameAndFriends($episode: Episode) {
            hero(episode: $episode) {
                name
                friends {
                    name
                }
            }
        }
    ```
- We can pass in default values and once a variable is passed in it will override the default value
    - When a default value is passed the query is called without passing variables
    - Useful if we want to render some initial data and later have it change on the fly
    - Example:
    ```graphQL
    query HeroNameAndFriends($episode: Episode: "JEDI) {
        hero(episode: $episode) {
            name
            friends{
                name
            }
        }
    }
    ```

### Directives

- Kind of like variables but they dynamically change the structure and shape of our queries
- A directive is either inclusive or exclusive
- There are only two directives in GraphQL and they are: ```@include(if: BOOL) @skip(if: BOOL)```
- Directives can be attached to fields and fragments
- Example: 
    ```graphql
    query Hero($episode: Episode, $withFriends: Boolean!)
        hero(episode: $episode) {
            name
            firends @include(if: $withFriends) {
                name
            }
        }
    }
    ```
- Every query the above will give us a hero's name in whatever episode we pass in
- If the $withFriends variable is true it will also include a list of the hero's friend's names alongside the hero's name
- Directives essentially allow us to add or remove fields in our queries

### Mutations

- Allow us to (guess what!?) mutate data!
- Technically any query can mutate data but it's best practice to explicitly use our operation type ```mutation``` to indicate a data write
- Mutations return an object so we can ask for certain nested fields to fetch the new state of data after an update
- Example:
    ```graphQL
    mutation CreateReviewForEpisode($ep: Episode!, $review: ReviewInput!) {
        createReview(episode: $ep, review: $review) {
            stars
            commentary
        }
    }
    ```
- The ```createReview``` field returns the stars and commentary fields of the newly created review
- The review variable is an *input object type* which is a special kind of object type that can be passed in as an argument

## Schemas and Types

- The most basic component of a GraphQL schema is what's known as an object type
- It's simply what the object that you fetch from queries will look like and what fields it has
- Example:

    ```graphQL
    type Character {
        name: String!
        appearsIn: [Empire]!
    }
    ```

- The ```String``` above is a scalar type in GraphQL (built in)
    - These are types that resolve to a single clear object and can't have sub-sections
- The ```!``` above represents a *non-nullable field*, we will always get a value when we query this field
- The ```[]``` represent an array of objects
- Another example:

```graphql
type Starship {
    id: ID!
    name: String!
    length(unit: LengthUnit = METER): Float
}
```

- All arguments are named and are passed by name specifically
- In the above example our field ```length``` takes one argument named ```unit```
    - The ```LengthUnit = METER``` is a default value
    - If a unit argument is not passed it will default to ```METER``` otherwise it will use the argument