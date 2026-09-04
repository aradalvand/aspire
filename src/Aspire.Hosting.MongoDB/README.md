# MongoDB hosting integration

Use this integration to model, configure, and orchestrate a MongoDB resource in an Aspire solution.

## Getting started

### Add the integration

From your AppHost directory, add the `Aspire.Hosting.MongoDB` integration with the Aspire CLI:

```bash
aspire add Aspire.Hosting.MongoDB
```

## Usage example

In the AppHost, add a MongoDB resource and reference it from another resource with either C# or TypeScript:

**C#**

```csharp
var db = builder.AddMongoDB("mongodb").AddDatabase("mydb");

var myService = builder.AddProject<Projects.MyService>()
                       .WithReference(db);
```

**TypeScript**

```typescript
const db = await builder.addMongoDB("mongodb").addDatabase("mydb");

const myService = await builder.addNodeApp("myService", "../my-service", "server.js")
                       .withReference(db);
```

### Replica set

A replica set groups several MongoDB servers into one logical resource, which is what enables transactions and change streams. Reference the replica set rather than any individual member:

**C#**

```csharp
var mongo1 = builder.AddMongoDB("mongo-1");
var mongo2 = builder.AddMongoDB("mongo-2");
var mongo3 = builder.AddMongoDB("mongo-3");

var replicaSet = builder.AddMongoDBReplicaSet("rs0")
                        .WithMember(mongo1)
                        .WithMember(mongo2)
                        .WithMember(mongo3);

var myService = builder.AddProject<Projects.MyService>()
                       .WithReference(replicaSet)
                       .WaitFor(replicaSet);
```

**TypeScript**

```typescript
const mongo1 = await builder.addMongoDB("mongo-1");
const mongo2 = await builder.addMongoDB("mongo-2");
const mongo3 = await builder.addMongoDB("mongo-3");

const replicaSet = await builder.addMongoDBReplicaSet("rs0")
    .withMember(mongo1)
    .withMember(mongo2)
    .withMember(mongo3);

const myService = await builder.addNodeApp("myService", "../my-service", "server.js")
    .withReference(replicaSet)
    .waitFor(replicaSet);
```

A single member is enough if all you need is transactions and change streams rather than redundancy. A replica set holds at most 50 members, the first seven of which vote in elections; the rest join as non-voting members that still carry a full copy of the data.

## TLS

A MongoDB server serves TLS whenever an HTTPS/TLS certificate is available for it, which by default is the ASP.NET Core developer certificate. `WithoutHttpsCertificate()` opts out and `WithTlsMode()` chooses how strict the server is about TLS on incoming connections. The connection string reports this through a `tls=true` flag that is resolved when the connection string is read, so consumers pick it up automatically.

The developer certificate is issued for `localhost`, so a consumer running on the host validates it without any further configuration. A consumer running in a container is a different matter: it reaches the server by its resource name on the container network, which is not a name the certificate carries, so its TLS handshake fails host name validation. Until certificates covering container network names are available, a containerized consumer of a TLS-enabled MongoDB server has to be configured to relax host name validation, or the server has to opt out of TLS with `WithoutHttpsCertificate()`.

## Connection Properties

When you reference a MongoDB resource using `WithReference`, the following connection properties are made available to the consuming project:

### MongoDB server

The MongoDB server resource exposes the following connection properties:

| Property Name | Description |
|---------------|-------------|
| `Host` | The hostname or IP address of the MongoDB server |
| `Port` | The port number the MongoDB server is listening on |
| `Username` | The username for authentication |
| `Password` | The password for authentication (available when a password parameter is configured) |
| `AuthenticationDatabase` | The authentication database (available when a password parameter is configured) |
| `AuthenticationMechanism` | The authentication mechanism (available when a password parameter is configured) |
| `Uri` | The connection URI, with the format `mongodb://{Username}:{Password}@{Host}:{Port}/?authSource={AuthenticationDatabase}&authMechanism={AuthenticationMechanism}` |

### MongoDB database

The MongoDB database resource combines the server properties above and adds the following connection property:

| Property Name | Description |
|---------------|-------------|
| `DatabaseName` | The MongoDB database name |

### MongoDB replica set

The MongoDB replica set resource exposes the following connection properties. It has no single `Host` and `Port`, because clients discover the members through the seed list carried in the `Uri`:

| Property Name | Description |
|---------------|-------------|
| `Username` | The username for authentication, shared by every member of the replica set |
| `Password` | The password for authentication, shared by every member of the replica set |
| `AuthenticationDatabase` | The authentication database |
| `AuthenticationMechanism` | The authentication mechanism |
| `ReplicaSetName` | The name of the replica set |
| `Uri` | The connection URI, with the format `mongodb://{Username}:{Password}@{Host1}:{Port1},{Host2}:{Port2}/?replicaSet={ReplicaSetName}&authSource={AuthenticationDatabase}&authMechanism={AuthenticationMechanism}` |

Aspire exposes each property as an environment variable named `[RESOURCE]_[PROPERTY]`. For instance, the `Uri` property of a resource called `db1` becomes `DB1_URI`.

## Additional documentation

* https://aspire.dev/integrations/gallery/
* https://aspire.dev/integrations/databases/mongodb/mongodb-host/

## Feedback & contributing

https://github.com/microsoft/aspire
