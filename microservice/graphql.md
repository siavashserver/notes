---
title: GraphQL
---

## Introduction

GraphQL is a query language and runtime for APIs. It lets clients _specify
exactly what data they need_, avoiding both over‑fetching and under‑fetching.
It's strongly typed, supports introspection, and combines querying, mutating,
and real‑time subscriptions via a schema.

## Communication Methods

### Query

Read-only operations.

```graphql
query GetBook($id: ID!) {
  book(id: $id) {
    id
    title
    pages
    author {
      name
    }
  }
}
```

### Mutation

Used for creating/updating/deleting data.

```graphql
mutation CreateBook($title: String!, $pages: Int!) {
  createBook(title: $title, pages: $pages) {
    id
    title
  }
}
```

### Subscription

Real-time updates via a long-lived connection (WebSockets or SSE).

```graphql
subscription OnBookAdded {
  bookAdded {
    id
    title
    author {
      name
    }
  }
}
```

## GraphQL vs REST

| Feature           | REST                           | GraphQL                                                    |
| ----------------- | ------------------------------ | ---------------------------------------------------------- |
| Endpoint          | Multiple fixed endpoints       | Single endpoint handling all queries & mutations           |
| Fetching          | Over-/under-fetching common    | Client selects the exact fields needed                     |
| Real-time support | Polling or SSE separately      | Subscriptions via WS or SSE                                |
| Versioning        | Versioned URIs (/v1/)          | Fields deprecated but schemas stay versionless             |
| Caching           | HTTP caching built-in          | More complex—can utilize query persistence or cache layers |
| Schema & typing   | Generally loose, often untyped | Strongly typed schema with introspection                   |
| Ecosystem         | Mature, simple tooling         | Requires more setup, steep learning curve                  |

---

## Sample Code

### Schema

```csharp
public class Book { public int Id; public string Title; public Author Author; }
public class Author { public int Id; public string Name; }

public class Query {
  public Book GetBook(int id, [Service] BookDbContext db) => db.Books.Include(b=>b.Author).FirstOrDefault(b=>b.Id==id);
}

public class Mutation {
  public async Task<Book> CreateBook(string title, int pages, [Service] BookDbContext db) {
    var book = new Book { Title=title, Pages=pages };
    db.Add(book); await db.SaveChangesAsync(); return book;
  }
}

public class Subscription {
  [Subscribe]
  [Topic]
  public Book BookAdded([EventMessage] Book b) => b;
}
```

### Startup Configuration

```csharp
services.AddGraphQLServer()
  .AddQueryType<Query>()
  .AddMutationType<Mutation>()
  .AddSubscriptionType<Subscription>()
  .AddInMemorySubscriptions();
app.UseWebSockets();
app.UseRouting();
app.UseEndpoints(endpoints => endpoints.MapGraphQL());
```

### Publishing Subscriptions

```csharp
// In mutation after saving:
pubSub.Publish(nameof(Subscription.BookAdded), book);
```

### JavaScript Client

```javascript
import { ApolloClient, InMemoryCache, gql, split } from '@apollo/client';
import { createClient } from 'graphql-ws';
import { GraphQLWsLink } from '@apollo/client/link/subscriptions';
import { HttpLink } from '@apollo/client';
import { getMainDefinition } from '@apollo/client/utilities';

const httpLink = new HttpLink({ uri: '/graphql' });
const wsLink = new GraphQLWsLink(createClient({ url: 'ws://localhost:5000/graphql' }));

const splitLink = split(
  ({ query }) => {
    const def = getMainDefinition(query);
    return def.kind === 'OperationDefinition' && def.operation === 'subscription';
  },
  wsLink,
  httpLink
);

const client = new ApolloClient({ link: splitLink, cache: new InMemoryCache() });

// Usage
client.query({ query: gql`{ book(id:1){title author{name}}}` });
client.mutate({ mutation: CREATE_BOOK_MUTATION, variables:{...} });
client.subscribe({ query: BOOK_ADDED_SUBSCRIPTION }).subscribe({
  next({ data }) { console.log('New book:', data.bookAdded); }
});
```

---

## Interview Questions

### How does caching work in GraphQL?

Client-side uses tools like Apollo Client cache; server-side caching can use
persisted queries or directive-based strategies. REST HTTP caching doesn't
directly map.

### How to version a GraphQL API?

Avoids versioned endpoints—use non-breaking schema changes, deprecate fields
using `@deprecated`, or add new types, keeping backward compatibility.

### Authentication & Error Handling?

Auth via HTTP headers (JWT etc) outside schema; in resolvers, check permissions.
Errors returned in _errors_ response alongside data (no HTTP codes).

### What is introspection?

A feature where clients can query the schema itself (types, fields, directives).
Enables self-documenting APIs and tooling like GraphiQL.
