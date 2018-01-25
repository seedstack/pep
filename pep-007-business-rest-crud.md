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
___
### Declaration 

The Idea is to introduce a annotation, which in presence of [@DtoOf](https://github.com/seedstack/business/blob/master/specs/src/main/java/org/seedstack/business/assembler/DtoOf.java) will mark that DTO to be published automatically on a rest endpoint with CRUD capabilities.

The proposed interface is 
```Java
@Retention(CLASS)
@Target(TYPE_USE)
public @interface AutoRest {
    boolean create() default true;
    boolean update() default true;
    boolean delete() default true;
    boolean query() default true;
    String restEndpoint() default "";
}

```` 
Such annotation would have a serie of optional parameters to tweak 
* Which CRUD verbs would be available
* URL where it will be published

Example:
To generate an endpoint at http://localhost/base-api-path/CustomFoo which allows Query/Update/Create but not Delete the code that should be put on place would be like:
```Java 
@DtoOf(Foo.class)
@AutoRest(delete=false,restEndpoint="CustomFoo")
public class FooRepresentation{
//DTO Body...
}
```
## Implementation

The proposal is to create a new module, pretty much like [ModelMapper Addon](https://github.com/seedstack/modelmapper-addon) that depends on business-core and seed-web that handles the recognization of ``@AutoRest`` and registers the path as an rest endpoint on startup
