---
title: How to use Slinky
description: Slinky is a library for quick development in Spring Boot.
date: "2022-01-09"
tags:
- Spring Boot
- Slinky
---
## Using slinky's generic classes

The following project is available at [paulo-e/slinky-kotlin-starter](https://github.com/paulo-e/slinky-kotlin-starter).

Having in mind our simple model "`Todo`" below, this is how you would go about creating a simple CRUD for this entity.

```kotlin
@Entity
@Table(name = "todos")
data class Todo(
	@Id
	@GeneratedValue(strategy = GenerationType.AUTO)
	override var id: Long?, // 2
	var name: String?,
	var description: String?,
	var completed: Boolean?
) : IGenericEntity<Long> // 1
```

1. Here we extend `IGenericEntity`. Inside slinky, this class is used as your entity. We can easily access your
   ID (`primary key`) inside slinky this way. We also provide `Long`, the class we are going to use as our ID.
2. Here we need to override our ID so slinky can access our actual ID. We use `@Id` and `@GeneratedValue` with `AUTO` to
   automatically generate our ID for us. But this is not required.

## Repository

```kotlin
@Repository
class TodoRepository(entityManager: EntityManager)
	: GenericRepository<Todo, Long>(Todo::class.java, entityManager)
```

Here we are creating a repository class that receives `EntityManager` through dependency injection and passes this value
to slinky's `GenericRepository`. Passing `Todo::class.java` or `Class<T>` is also required.

## Business

```kotlin
@Service
class TodoBusiness(repository: TodoRepository)
   : GenericBusiness<Todo, Long>(repository)
```

Inside our business (or service) class, we receive our repository (again, through dependency injection) and pass it to
slinky's `GenericBusiness` class.

## Controller

```kotlin
@RestController
@RequestMapping("/api/v1/todos") // 1
class TodoController(business: TodoBusiness)
   : GenericController<Todo, Long>(business) // 2
```

1. Here we provide the base mapping we want our controller to have. Supposing we are running our server at `localhost`
   with the port 8080, we are going to access our controller at `localhost:8080/api/v1/todos`.
2. One more time, we are extending slinky's generic class to receive basic functionality.

## Done

Provided you configured your `application.properties`, all you need to do is to access your server URL and use the
required HTTP verbs to access your resource. Our controller will accept:

- `GET /api/v1/todos`
    - Retrieves all resources from our database.
- `GET /api/v1/todos/:id`
    - Retrieves one resource.
- `GET /api/v1/todos/:id/completed`
    - In our case, this is going to return us the value of this Todo's completed status.
- `POST /api/v1/todos`
    - Creates a new resource
- `PUT /api/v1/todos/:id`
    - Updates a resource. The whole resource needs to be provided. Missing fields will be set to `null`.
- `PATCH /api/v1/todos/:id`
    - Updates a resource. Missing fields will be ignored and not set to `null`.
- `HEAD /api/v1/todos/:id`
    - Returns `HTTP 200` if the resource exists. `HTTP 404` if it doesn't.
- `OPTIONS /api/v1/todos`
    - Returns the verbs this server accepts inside an HTTP header.
