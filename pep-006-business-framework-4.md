    PEP: 006
    Title: Business framework 4.0
    Author: Adrien LAUER <adrien.lauer@mpsa.com>  
    Status: draft  

# Abstract

The goal here is two-fold:
* Refactor business framework legacy parts into something more modern,
* And take advantages of Java 8 features everywhere it makes sense.

# Motivation

We want to improve the developer experience of using the business framework. For this we must fix long-standing oddities and use newer features of the language where it makes sense. Functional upgrades of existing patterns can also be considered.

# Specification

## GenericRepository

The `GenericRepository` interface is a left-over from older versions and is replaced in purpose by the `Repository` interface. 
It should be removed.

## GenericFactory / Factory

To be consistent with other patterns, `GenericFactory` should be renamed `Factory` and `Factory` should be removed. The `create(...)` method should have a default implementation in the interface so it won't need to be implemented by custom factories.

## Repository

The `Repository` interface should be reworked to be more inline with the original spirit of the repository pattern:

> A repository represents all objects of a certain type as a conceptual set (usually emulated). It acts like a collection, except with more elaborate querying capability. […] For each type of object that needs global access, create an object that can provide the illusion of an in-memory collection of all objects of that type.

The proposed interface is the following:

```java
@DomainRepository
public interface Repository<A extends AggregateRoot<K>, K> {

    void add(A aggregate);

    Stream<A> get(Specification<A> specification, Options... options);

    Optional<A> get(ID id);

    boolean contains(Specification<A> specification);

    boolean contains(ID id);

    boolean contains(A aggregate);

    long count(Specification<A> specification);

    long size();

    boolean isEmpty();

    long remove(Specification<A> specification);

    boolean remove(ID id);

    boolean remove(A aggregate);

    void update(A aggregate);

    void clear();

}
```

## FluentAssembler

### Make FluentAssembler accept streams

Fluent assembler should be able to use streams instead of lists for assembling multiple aggregates/dtos. Combined with the streaming ability of repositories, this would enable advanced use-cases:

```java
public class SomeClass {
    @Inject
    private Repository<Customer, CustomerId> customerRepository;
    @Inject
    private FluentAssembler fluentAssembler;
    
    public void someMethod() {
        fluentAssembler
            .assemble(customerRepository
                .aggregates(CustomerSpecifications.RecentClients, Sorting.Natural)
                .parallelStream()
                .filter(customer -> customer.getGender() === CustomerGender.FEMALE))
            .to(CustomerDTO.class)
            .collect(toList());
    }
}
```

## Assembler interface

Sometimes only one-way assembling is needed. The current `Assembler` interfaces forces to implemented both ways and should be splitted into two separate interfaces: `DTOAssembler` and `AggregateAssembler`.

## Assembler parameters

Parameters for assembler methods should be reversed, with the source as first parameter and the target as second parameter. 

## Pagination

Pagination should be reworked based on new capabilities of repositories, notably the ability to return streams from specifications.
A pagination DSL could help developpers to extract a part of the results automatically.

### Page pagination

```java
public class SomeClass {
    @Inject
    Paginator paginator;
    @Inject
    FluentAssembler fluentAssembler.
    @Inject @Jpa
    Repository<MyAggregate, String> myRepository;
        
    public void someMethod() {        
        Page<MyDTO> dtoPage = fluentAssembler.assemble(
            paginator
                .repository(myRepository)
                .options(...)
                .page(7)
                .size(10)
                .paginate(someSpec)) // or .paginate()
            .with(ModelMapper.class)
            .to(MyDTO.class);
    }
}
```

### Chunk pagination

```java
public class SomeClass {
    @Inject
    Paginator paginator;
    @Inject
    FluentAssembler fluentAssembler.
    @Inject @Jpa
    Repository<MyAggregate, String> myRepository;
        
    public void someMethod() {        
        Chunk<MyDTO> dtoChunk = fluentAssembler.assemble(
            paginator
                .repository(myRepository)
                .options(...)
                .offset(70)
                .limit(10)
                .paginate(someSpec)) // or .paginate()
            .with(ModelMapper.class)
            .to(MyDTO.class);
    }
}
```

### Key-based chunk pagination

```java
public class SomeClass {
    @Inject
    Paginator paginator;
    @Inject
    FluentAssembler fluentAssembler.
    @Inject @Jpa
    Repository<MyAggregate, String> myRepository;
        
    public void someMethod() {        
        Chunk<MyDTO> dtoChunk = fluentAssembler.assemble(
            paginator
                .repository(myRepository)
                .options(...)
                .after("insertDate", someDate).limit(10)
                .paginate(someSpec)) // .paginate()
            .with(ModelMapper.class)
            .to(MyDTO.class);
    }
}
```
