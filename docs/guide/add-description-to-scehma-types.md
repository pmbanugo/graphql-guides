# Add Description to your Schema Types

Your schema should be as self-descriptive as possible. There will be times when you want to add documentation describing a type, field, or other definitions you can have in GraphQL. To achieve that, you'll use what is called `description` in GraphQL. GraphQL descriptions are provided alongside their definitions and made available via introspection. All types in the introspection system provide a `description` field of type `String` to allow you to publish documentation and they're defined using Markdown.

For example, a simple schema which returns a list of `Book` type can be described as follows:

```graphql
type Book {
  chapters: Int!
  pages: Int!
  title: String!
}

type Query {
  """
  Returns a Book where the argument value is part of the Title of the Book
  """
  book(
    """
    text to search for
    """
    title: String
  ): Book

  """
  Returns a list of `Book` type
  """
  books: [Book]
}
```

## Schema-First

There are different ways to implement this in Hot Chocolate. If you're using schema-first, the schema parser supports the inclusion of description string:

```csharp
SchemaBuilder.New()
    .AddDocumentFromString(@"
        type Book {
        chapters: Int!
        pages: Int!
        title: String!
        }

        type Query {
            """
            Returns a Book where the argument value is part of the Title of the Book
            """
            book(
                ""text to search for""
                title: String
            ): Book

            """
            Returns a list of `Book` type
            """
            books: [Book]
        }")
    .AddResolver("Query", "book", () => null)
    //TODO: Add other resolvers
    .Create();
```

## Using Attributes

If you build your schema using code-first, you can use attributes to decorate the the types and Hot Chocolate will pick it up. Here's how you'll define the `Query` type in our example in code-first:

```csharp
public class Book
{
    [GraphQLNonNullType]
    public string Title { get; set; }
    public int Pages { get; set; }
    public int Chapters { get; set; }
}

public class Query
{
    List<Book> books = new List<Book>{
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

    [GraphQLDescription("Returns a list of `Book` type")]
    public IEnumerable<Book> GetBooks() => books;

    [GraphQLDescription("Returns a Book where the argument value is part of the Title of the Book")]
    public Book GetBook([GraphQLDescription("text to search for")]string title) => books.Find(b => b.Title.Contains(title));
}
```

You'll need to reference the `HotChocolate` namespace in order to use that attribute (i.e `using HotChocolate`).

## Using Fluent API

If you don't want to sprinkle any of Hot Chocolate's attributes on your domain objects, you can use schema types. You could do this by creating a new class that extends any of Hot Chocolate's schema type. With that you can re-write the example to the code below:

```csharp
public class Query
{
    //... code goes here

    public IEnumerable<Book> GetBooks() => books;
    public Book GetBook(string title) => books.Find(b => b.Title.Contains(title));
}

public class QueryType : ObjectType<Query>
{
    protected override void Configure(IObjectTypeDescriptor<Query> descriptor)
    {
        descriptor.Field(f => f.GetBooks()).Description("Returns a list of `Book` type");
        descriptor.Field(f => f.GetBook(default))
            .Description("Returns a Book where the argument value is part of the Title of the Book")
            .Argument("title", argDescriptor => argDescriptor
                    .Description("text to search for"));
    }
}
```

You'll need to reference the `HotChocolate.Types` namespace by putting `using HotChocolate.Types;` at the top of the file.

## Using XML Documentation

You can also use XML documentation comments to add descriptions but I prefer to use attributes or the fluent configuration. Read the [docs](https://hotchocolate.io/docs/schema-descriptions#xml-documentation) if you'd like to see how to configure your project so that it uses XML documentation comments as descriptions.
