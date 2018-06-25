# Sync Server

## Terminology

**GraphQL** - A data query language

**IDL/SDL** - Interface/Schema Definition Language. The language a GraphQL Data Schema is written in.

**Client** - Any device or browser that sends a GraphQL Query or Mutation request to the server

**Data Source** - A source for loading/saving data e.g. a Postgres database, a REST API, an OpenWhisk action

**Resolver Mapping** - The definition for how to load/save data to a Data Source based on parameters sent by a Client, and how to format it before returning it to the client. All mapping defintions are compiled and executed as a Handlebars template. This allows for some logic when defined the request & repsonse mapping e.g. using the `toJson` help to stringify the result of an SQL query.

**Resolver Request Mapping** - The defintion for how to load/save data to a Data Source based on parameters sent by a Client e.g. a SQL query, a HTTP endpoint, method & body.

**Resolver Response Mapping** - The definition for how to format data retrieved from a Data Source before returning it to the client e.g. simply stringify an array of objects.

**Connector** - The combination of a Data Source and Resolver Mapping capabilities e.g. a Postgres Connector would allow a developer to connect to a Postgres database, and define SQL queries to execute when particular GraphQL Queries or Mutations are called.

More information about the GraphQL terminology and schema can be found at https://graphql.org/learn/schema/

## PostgreSQL Connector

### Data Source

The defintion for a PostgreSQL Data Source will have the following parameters:

* Database Server Host
* Database Server Port
* Database Name
* Database Username
* Database Password

Table names will not be configured as part of the Data Source for Postgres.
Instead, the table will be determined by the SQL query defined in the Resolver Request Mapping.

### Resolver Request Mapping

This is the defintion of an SQL query to execute.
Any parameters defined in the Data Schema for the Query or Mutation being executed will be made available on the `conext` object as `context.arguments`.
For example, to find a single record by id, use the following:

```
// readNote(id: String): Note

SELECT id, title, content, timestamp FROM \"Notes\" WHERE id='{{context.arguments.id}}';
```

To update a row, use the following:

```
// updateNote(id: String,title: String): Note

UPDATE \"Notes\" SET \"title\"='{{context.arguments.title}}' WHERE \"id\"='{{context.arguments.id}}' RETURNING *;
```

### Resolver Response Mapping

This is the defintion for how to format the result of a SQL query to match the expected GraphQL Data Schema for that Query or Mutation.
The result of the SQL query will be made available on the `context.result` object.
The `result` field will always be an `array`.
If a list of objects is expected in the Schema rsponse, this can be mapped as follows:

```
{{toJSON context.result}}
```

If a max of 1 result is expected, e.g. finding a record by id, this can be mapped as follows:

```
{{toJSON context.result.[0]}}
```

## HTTP Connector

### Data Source

The defintion of a HTTP Data Source will have the following parameters:

* Protocol i.e. HTTP or HTTPS
* Host e.g. www.example.com
* Port
* Base path e.g. /myapi/v1
* Headers (optional)
* Basic Auth Username (optional)
* Basic Auth Password (optional)

### Resolver Request Mapping

This is the defintion of a HTTP request to make for a specific Query or Mutation.
It will have the following parameters in conjunciton with the Data Source parameters to make a request.

* Path (Added to the end of the Base path defined in the Data Source. Can include query parameters)
* Additional Headers (optional)
* Method e.g. GET, POST, PUT
* Request Body (Optional)

These will be defined as a JSON object, but will be treated as a string.
This means the below defintion would actually be stored as the string beneath

JSON version
```
{
  "path": "/notes",
  "method": "GET",
  "headers": {
    "accept": "application/json"
  }
}
```

String version
```
{\n  \"path\": \"/notes\",\n  \"method\": \"GET\",\n  \"headers\": {\n    \"accept\": \"application/json\"\n  }}
```

By treating the defintion as a string, it allows for Handlebars syntax to be added anywhere in the JSON object.
For example, the below JSON defintion could be used for a simple find Query.

```
{
  "path": "/notes?id={{context.arguments.id}}",
  "method": "GET",
  "headers": {
    "accept": "application/json"
  }
}
```

### Resolver Response Mapping

This is the defintion for how to format the result of the HTTP Request.
The response body will be made available on `context.result`.
It's type depends on the `Accept` header in the request.
Initially, only `application/json` will have special behaviour, meaning the response body is parsed into a JSON object.
All other `Accept` header values will be ignored, in which case `context.result` will be the plain response body as a string.

If the result from the HTTP call is exactly as you need it, you can simply dump out the result.

```
{{toJSON context.result}}
```

If you need to iterate through the response as a JSON array, mapping fields, you can do something like below.
In this example, each item has a number of fields, one of which is `name`.
However, this needs to be changed to `firstname` to match our schema.

```
TODO: Helper for mapping an array
TODO: Is this a reasonable use case for response mapping? Maybe filtering is better e.g. omitting records that don't have a paritcular field set

{{toJSON mappedResult}}
```


## In Memory Database Connector

### Data Source

The In Memory Data Source is intended only for development purposes.
It should *not* be used for production data.
Any stored data in the In Memory Data Source will be wiped whenever the server restarts.

The defintion of an In Memory Data Source will have the following parameters:

* Collection Name
* Datastore Options (TBD which options)

The In Memory Data Source will be implemented using the [nedb](https://github.com/louischatriot/nedb) library.
Each Type defined in the Data Schema will typically need a new In Memory Data Source for the Collection that will store the data.
This is different from the Postgres Data Source, which only requires 1 Data Source defintion to connect to the database.
The reason for this difference is because the Postgres Request Mapping allows defining an entire SQL Query (including the table(s) to get data from).
While the In Memory Data Source allows defining an operation and parameters against an individual collection.

The Datastore Options that can be configured are based on what options are available for the nedb DataStore class.
For example, there is a `timestampData` option to auto create/update `createdAt` and `updatedAt` fields.

### Resolver Request Mapping

This is the defintion of an In Memory request for a specific Query or Mutation.
It will have the following parameters.

* Operation e.g. find, insert, update, remove
* Query (Only for some operations. See the nedb API)
* Document (Only for some operations. See the nedb API)
* Options (Only for some operations. See the nedb API)

For example, to find a single record by id:

```
{
  "operation": "find",
  "query": {
    "id": "{{context.arguments.id}}"
  }
}
```

Like all other request mappings, this will be treated as a string for Handlebars processing before being parsed as JSON.

To create a record, here's an example:

```
{
  "operation": "insert",
  "doc": {
    "title": "{{context.arguments.title}}",
    "content": "{{context.arguments.content}}"
  }
}
```


### Resolver Response Mapping

For In Memory responses, the `context.result` object will be result object passed back from the corresponding nedb API.
For example, for an `insert`, the created document is returned. This can be mapped as follows:

```
{{toJSON context.result}}
```

However, for a `remove` operation, the nedb API returns a `numRemoved` value. This could be mapped to an Int in the Data Schema.

```
// removeNote(id: String) Int
{
  "numRemoved": {{context.result}}
}
```

## Metrics

The primary goals of Data Sync metrics are:

* seeing resource usage of the Data Sync service over time
* summary metrics about the number of queries/mutations resolved by the server
* getting an insight into what Clients are connected/executing queries over time, the number of queries/mutations, which queries/mutations, and the size of data
  * Clients connected for Subscriptions, over websocket, may be a good way of determining when a client is connected and condition of their network
* being able to get Client specific information for debugging purposes e.g. a count of pending mutations that have built up while offline (depends on an offline mutation queueing feature)

The above is not intended to limit what metrics to capture, but more as a starting point to discover what metrics are valueable.

### Client Metrics

There are 2 potential approaches to getting Client Metrics from a device to the Metrics service. The first approach is to use the MetricsService module in the AeroGear SDK to send metrics directly to the Metrics service. This would mean the Sync module in the SDK would need to call the Metrics service every so often.
This is not a great approach as the device may not be online all the time.

The second approach is to piggyback on any GraphQL requests, adding metrics data to the request context. The data will need to be limited in size to avoid unnecessary network overhead. Using simple counters that are maintained on the Client will help keep the payload size small. For example, the Sync module in the SDK maintains a specific set of counters as the App interacts with it.
Whenever a Sync request (query/mutation) is made to the server, the latest values for this set of counters is sent along.
On the server, these counter values are exposed via a prometheus endpoint.
Prometheus polls the Data Sync service for these values, allowing them to be queried and visualised in Grafana.

Visualisation of Client Metrics will ultimately be based on any Client specific metrics sent with each GraphQL request, and other Client specific metrics during the query/mutation resolver execution on the Server i.e. whenever the Server is resolving data for that Client. This will form the basis for some initial Grafana queries and dashboards. However, this will likely need more thought and discussion closer to the time after a steel thread is acheived.

### Server Metrics

Server Metrics that are not specific to a Client can also be gathered and visualised.
This can include:

* memory & CPU usage
* resolver timings, broken down into request & response times

Other metrics can be added based on their merit of being valueable to the administrator/operator of Data Sync.

## Subscriptions (for Realtime Client updates)

In a typical GraphQL server, a Subscription has a resolver function that notifies any connected client if a change happens.
For example, if you have a mutation for creating a 'Note', and also want to listen for whenever any Client creates a 'Note', the schema might look like this:

```
type Note {
  id: ID
  content: String
}

type Mutation {
  createNote(content: String): Note
}

type Subscription {
  noteCreated: Note
}
```

The `createNote` resolver function might use an in-memory pub/sub system/library to publish an event whenever it finishes successfully.
In turn, the `noteCreated` resolver would listen for that event, and notify any connected Client about the event.

As the Data Sync server is a generic GraphQL server, it has no way to know how Subscriptions relate to Queries and Mutations.
However, with the right schema syntax, the developer can inform the server of that link by using a directive:

```
type Subscription {
	noteCreated: Note
		@datasync_subscribe(mutations: ["createNote"])
}
```



## Authentication

x

## Authorization

x

## Auditing (Logs)

x

## Webhooks

x

## Conflict Resolution/Strategy

x