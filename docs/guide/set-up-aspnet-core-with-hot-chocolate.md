# Set up an ASP.NET Core, GraphQL Project

GraphQL has been gaining wide adoption as a way of building and consuming Web APIs. GraphQL is a specification that defines a type system, query language, and schema language for your Web API, and an execution algorithm for how a GraphQL service (or engine) should validate and execute queries against the GraphQL schema. With GraphQL, varying clients can query for the type of data they need.

In this guide, I'll show you how to set up a GraphQL server in ASP.NET Core. You're going to use Hot Chocolate, which has a set of libraries for building GraphQL API in .NET.

## Creating the ASP.NET Core project

You’re going to create an ASP.NET Core application to host the GraphQL server. Open Visual Studio or JetBrains Rider and create a new ASP.NET Core project with the **Empty** template, or open your command-line application and follow the instructions below if you use VS Code:

- Run the command `mkdir <insert-project-name> && cd <insert-project-name> && dotnet new web`. You should replace `<insert-project-name>` with your preferred project name.

- Run the command `dotnet add package HotChocolate.AspNetCore` to install Hot Chocolate's ASP.NET Core NuGet package. _Note: the version of used here is 10.4.0 which is the latest at the time of this writing._

The instructions above used the dotnet CLI to add packages from NuGet. If you use Visual Studio and want to use the NuGet Package Manager console, run the command `Install-Package HotChocolate.AspNetCore -Version 10.4.0`.

At this point, you have a basic ASP.NET Core project with the necessary dependencies needed to configure the GraphQL server and serve the API. Let's move on to designing the schema.

## Designing the Schema

We're going to use a schema with one query operation to retrieve a list of books. For that, you'll need a class that represents a book. Add a class named **Book.cs** to your project

```csharp
using HotChocolate;

public class Book
{
    [GraphQLNonNullType]
    public string Title { get; set; }
    public int Pages { get; set; }
    public int Chapters { get; set; }
}
```

In the code snippets you just added, you saw the attributes `[GraphQLNonNullType]`. It is one of the attributes from the Hot Chocolate library and you used it to specify the type for some fields. The `[GraphQLNonNullType]` attribute is used to indicate that a field is non-nullable. For example, you used it to indicate that the title field of the Book type cannot be null. Hot Chocolate will infer the scalar type from the .NET type associated with the property.

The next step will be to define the _Query_ type which will include the root operation for getting the list of books. Add a class named **Query.cs** and paste the code below in it.

```csharp
using HotChocolate;

public class Query
{
    [GraphQLNonNullType]
    public IEnumerable<Book> GetBooks()
    {
        var books = new List<Book>{
            new Book {
                Title = "GraphQL Schema Design for the Enterprise",
                Chapters = 4,
                Pages = 450
            },
            new Book {
                Title = "Introductory tutorial to GraphQL",
                Chapters = 9,
                Pages = 1050
            }
        };

        return books;
    }
}
```

The method in this class will be translated to a field in the _Query_ root type and the method definition is the resolver function for this type. By convention, Hot Chocolate removes the word "Get" from the method name and uses the remainder of the word for the field name. When this resolver is executed, it'll return the list of books you see above.

## Setting up the GraphQL Middleware

So far, you’ve added code that defines the GraphQL schema and its resolvers. The next thing you’re going to do is configure the GraphQL ASP.NET Core middleware. Open **Startup.cs** and add the using statements below:

```csharp
using HotChocolate;
using HotChocolate.AspNetCore;
```

Go to the `Configure` method and register Hot Chocolate's GraphQL middleware by adding `app.UseGraphQL();` to the method.

You will define your schema in Hot Chocolate with the `SchemaBuilder`. To do that, go to the `ConfigureServices` method and add the code below to it.

```csharp
services.AddGraphQL(SchemaBuilder.New()
    .AddQueryType<Query>()
    .ModifyOptions(o => o.RemoveUnreachableTypes = true)
    .Create());
```

If you run the application, it should give you a GraphQL schema definition as follows:

```
type Book {
  chapters: Int!
  pages: Int!
  title: String!
}

type Query {
  books: [Book!]!
}
```

## What's Next?

You can view the schema and run queries from tools like GraphQL Playground or GraphiQL. There's a separate [page](#) for setting up GraphQL Playground and running queries in it. You should check out the guide on [query root type](#) to learn more about queries and working with Entity Framework.
