    PEP: 007
    Title: Business REST CRUD
    Author: Xián BOULLOSA <xianbtrigo@gmail.com>
    Status: draft  

# Abstract

The goal of this PEP is to propose a default REST resource implementation that would expose any marked DTO as a REST CRUD resource. 
This resource would support hypermedia and pagination.

# Motivation

We want to speed-up the development of business code with SeedStack. Although we don't want to reduce REST API design to 
simple CRUD operations, it is nonetheless a big part of the development work. This proposal would significantly reduce this work
for the developer.

# Specification

### Custom resource

A base resource class implementing the CRUD behavior would be provided so it can be extended easily:

```java
public abstract class BaseCrudResource<A extends AggregateRoot<I>, I, D> {
    // base implementation of the CRUD API
}
```

By extending this class, the developer could:

* Specify class-level JAX-RS annotations (like `@Path`).
* Specify additional methods to manipulate the resource.

### Default resource

To allow the exposition of a CRUD API without writing any resource class, an annotation would be introduced. When combined with [@DtoOf](https://github.com/seedstack/business/blob/master/specs/src/main/java/org/seedstack/business/assembler/DtoOf.java), it would mark a DTO to be published automatically on a default CRUD resource. The proposed annotation is:

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.ANNOTATION_TYPE})
public @interface CrudResource {
    String value() default "";

    boolean create() default true;
    
    boolean read() default true;
    
    boolean update() default true;
    
    boolean delete() default true;
}
```` 

Such annotation would have a serie of optional parameters to tweak:

* Which CRUD verbs would be available.
* The path of the published resource. If the path is left empty, a name could be automatically derived by removing known suffixes from the class name (DTO, Dto, Representation, ...), converting the rest to snake case and pluralize the last word.

As an example, to generate an endpoint at http://localhost/base-api-path/custom-foo which allows Read/Update/Create but not Delete the code that should be put on place would be like:

```java 
@DtoOf(Foo.class)
@CrudResource(value= "/custom-foo", delete = false)
public class FooRepresentation{
    // DTO Body...
}
```

### Published API

For each resource the following API would be published:

* `GET /base-api-path/resource-path`: fetches all aggregates mapped as the representation. Supports pagination, filtering and sorting.
* `POST /base-api-path/resource-path`: creates an aggregate from the representation provided as request body.
* `DELETE /base-api-path/resource-path`: clear all aggregates corresponding to the representation.
* `GET /base-api-path/resource-path/{id}`: fetches the aggregate identified with `{id}` mapped as the representation.
* `PUT /base-api-path/resource-path/{id}`: fully updates the aggregate identified with `{id}` from the representation provided as request body.
* `DELETE /base-api-path/resource-path/{id}`: deletes the aggregate corresponding to the representation and identified with `{id}`.
* `PATCH /base-api-path/resource-path/{id}`: partially updates the aggregate identified with `{id}` from the partial representation provided as request body.

## Implementation

The proposal is to create a new module, pretty much like [ModelMapper Addon](https://github.com/seedstack/modelmapper-addon) that depends on business-core and seed-web that handles the recognization of `@CrudResource` and registers the path as an rest endpoint on startup.

