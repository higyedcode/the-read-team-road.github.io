# GraphQL API
- `enables users to specify exactly what fields they want in the response`
- Clients don't need to know API endpoints, one endpoint, ask for data -> GraphQL API takes care of retrieving the data for you.
## HOW IT WORKS ?
- 3 operations
    - Fetching data
    - Modifying data
    - Subscriptions (permanent connection where server continuously pushes data to client)
- It uses `objects (types), fields and relationships`
- **Only uses POST requests tot ONE ENDPOINT => response in JSON**

### GraphQl translation of REST APIs

- `GET requests`
    - `query` operation type (optional) + `name`(optional) + `data structure` + `arguments`(optional)
    ```js

    query myGETExampleName {
        getProduct(id: 123){
            name
            description
        }
    }
    ```
- `POST/PUT/DELETE requests`
    - **GraphQL Mutations**
    - same as `QUERIES`, but they take some input (`inline/variable`)

    ```js
    mutation exampleMutationName{
        createProduct(name: "TEST", listed:"yes"){
         id
         name
         listed   
        }
    }
    ```
- `GraphQL variables`
    - Taken from a `JSON-based VARIABLES dictionary `
    - Steps to use variables
        - Declare in the query header (after the name) - `$id: ID!`, `!` - required
        - Use the declared value in the query as argument - `id:$id`
        - Declare the variable value in the `Variables` dictionary
    ```js
    
     query myGETExampleName ($id: ID!) {
        getProduct(id: $id){
            name
            description
        }
    }

    Variables:
    {
        "id": 1
    }
    ```
- `Aliases`
    - GraphQL objects can't contain the same property/object twice
    - Use ALIASES to return the same object with different arguments
    - This can allow you to basiucally send `multiple GraphQL messages in one HTTP request`
    ```js
      #Valid query using aliases

    query getProductDetails {
        product1: getProduct(id: "1") {
            id
            name
        }
        product2: getProduct(id: "2") {
            id
            name
        }
    }
    ```
- `Fragments`
    - reusable parts of queries or mutations
    - reusable object / field grouping
    - can be included then in queries with `...` preceding it
    ```js
    # Example fragment
    fragment fragmentName on Product{
        id
        name
        listed
    }

     #Query calling the fragment

    query {
        getProduct(id: 1) {
            ...productInfo
            stock
        }
    }
    ```
- `SUBSCRIPTIONS`
    - implemented using Websockets
    - long-lived connection, small regular updates to big objects

- `INTROSPECTION`
    - BUILT IN FUNCTION THAT ENABLES TO QUERY SERVER FOR `SCHEMA INFORMATION`

#
# GraphQL vulnerabilities

## Finding GraphQL endpoints
- Use `universal queries`: `query {__typename}` -> should return `{"data":{"__typename": "query"}}`
    - Every GraphQL endpoint has a reserved field __typename that returns the queried object's type

- Use `blank body` -> should get a `query not present` error message!!!
- Common endpoint names
```js
/graphql
/api
/api/graphql
/graphql/api
/graphql/graphql
/graphql/v1
/api/v1
/api/graphql/v1
/graphql/api/v1
/graphql/graphql/v1

```
- Test `REQUEST HTTP METHODS`
    - Production GraphQL endpoints SHOULD ONLY ALLOW `POST + application/json`
    - Test for `GET` or `POST + application/x-www-form-urlencoded`
    - **Try UNIVERSAL QUERY + other HTTP methods**

## Exploiting unsanitized arguments
- Reference hidden objects by ID (ex: query for all -> returns object with id: 1,2,4 => query for (id: 3) reveals hidden object)

## Discovering schema information
- `INTROSPECTION + __schema on root`
- In REPEATER of a GraphQL API endpoint, right-click anywhere and select `GraphQL > Set introspection query` => adds it to the body of the request
- Use a `GRAPHQL visualizer` to interpret results (http://nathanrandal.com/graphql-visualizer/)
- If introspection is disabled, you can use `suggestions` to uncover the API functions (`Clairvoyance` - tool for suggestion GraphQL API mapping)

- MANUAL tests
```js
 #Introspection probe request

    {
        "query": "{__schema{queryType{name}}}"
    }
```
```js
#Full introspection query

    query IntrospectionQuery {
        __schema {
            queryType {
                name
            }
            mutationType {
                name
            }
            subscriptionType {
                name
            }
            types {
             ...FullType
            }
            directives {
                name
                description
                args {
                    ...InputValue
            }
            onOperation  #Often needs to be deleted to run query
            onFragment   #Often needs to be deleted to run query
            onField      #Often needs to be deleted to run query
            }
        }
    }

    fragment FullType on __Type {
        kind
        name
        description
        fields(includeDeprecated: true) {
            name
            description
            args {
                ...InputValue
            }
            type {
                ...TypeRef
            }
            isDeprecated
            deprecationReason
        }
        inputFields {
            ...InputValue
        }
        interfaces {
            ...TypeRef
        }
        enumValues(includeDeprecated: true) {
            name
            description
            isDeprecated
            deprecationReason
        }
        possibleTypes {
            ...TypeRef
        }
    }

    fragment InputValue on __InputValue {
        name
        description
        type {
            ...TypeRef
        }
        defaultValue
    }

    fragment TypeRef on __Type {
        kind
        name
        ofType {
            kind
            name
            ofType {
                kind
                name
                ofType {
                    kind
                    name
                }
            }
        }
    }
```
## Introspection query disabled bypass
- regex matching for `__schema` can be bypassed by appening special characters
    - ` `, `\n`, `,` are ignored by GraphQL but not regex
- Introspection might be disabled on POST; try `GET` or `POST + x-www-form-urlencoded`

## Bypassing rate limiting using aliases
- if rate limiting is done based on the nr of HTTP requests, then using aliases can enable you to send as many GraphQL messages in one request as you want
![alt text](image-18.png)

## CSRF via GraphQL
- if `NO csrf-token` + `content-type not validated` => **CSRF**
- `Can use GET requests or POST with x-www-form-urlencoded for CSRF attacks!!!`

#
#
# GraphQL labs
1. Accidental exposure of private GraphQL fields
- found GraphQL endpoint in Burp History
- Sent introspection query, interpreted query results
![alt text](image-14.png)
- Sent query for `getUser(id)` for id 1 -> got administrator username and password in plaintext
![alt text](image-15.png)
- Sent login mutation with the username and password and got the auth token in the cookie
![alt text](image-16.png)

2. Finding a hidden GraphQL endpoint
- do fuzzing on API endpoints 
    - found `/api POST` method not allowed, (WEIRD)
    - tried `GET /api` => `query not specified!` => GRAPHQL endpoint found
    - tried universal query to confirm: `?query={__typename}` => got typename query
    - tried `introspection insertion with BURP` => got blocked
    - added special chars (` `, `,`, `\n`) and the newline worked
    - `!!! Right click on introspection request => GraphQL => ADD TO SITEMAP!!!`
    - this adds all the available queries and mutations for you (`interpreted` & `ready 2 go!!!`)
    - Go to Target, site-map, /api
    - found 1 mutation, needed an id. Used the 1 query found to get id=3 for carlos
    - Deleted user with id=3 with mutation
    ![alt text](image-17.png)

3. Performing CSRF exploits over GraphQL
- For CSRF to work you have to have either
    - `GET allowed`
    - `x-www-form-urlencoded POST allowed`
- The 1st one blocked, 2nd one allowed here
- Burp CAN'T CHANGE THE REQUEST RIGHT AWAY
    - Either do a simple universal query first and then edit the query in the GraphQL tab (query={__typename})
    - Or just URL encode all characters of the requested query
    ![alt text](image-19.png)

# Preventing GraphQL attacks
- DISABLE `INTROSPECTION`
- DISABLE `suggestions`
- do not expose unintended fields publicly
#
- limit the `query depth` (nr of queries in query) of your API's queries (prevents bruteforce and DOS)
- Configure `operation limits` based on the nr of operations not HTTP requests
    - this limits the nr of `unique fields`, `aliases` you can use
- Implement `cost analysis on your API` - if it is too expensive, drop the request. This is realtime process
#
- Only accept POST + JSON
- Validate the content provided matches the `Content-Type`
- Use a secure `CSRF token mechanism`