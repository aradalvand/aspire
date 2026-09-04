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
