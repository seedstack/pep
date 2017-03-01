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

    Optional<A> get(K id);

    void add(A aggregate);
    
    void update(A aggregate);

    boolean remove(A aggregate);

    boolean removeByKey(K id);

    boolean contains(A aggregate);

    boolean containsKey(K id);    
    
    long clear();
    
    long size();

}
```

## MatchingRepository

The `MatchingRepository` adds methods to retrieve aggregates (or keys) that match a specification. These methods can be combined with unitary methods from `Repository` to delete or update multiple aggregates. 

```java
public interface MatchingRepository<A extends AggregateRoot<K>, K> extends Repository<A, K> {

    Stream<A> aggregates(Specification<A> specification, Sorting<A> sorting);

    Stream<K> keys(Specification<A> specification, Sorting<A> sorting);
    
}
```

## RangeRepository

The `RangeRepository` adds overloads of `MatchingRepository` methods that take a `Range` as third parameter to restrict the returned stream to a specific range.

```java
public interface RangeRepository<A extends AggregateRoot<K>, K> extends MatchingRepository<A, K> {

    Stream<A> aggregates(Specification<A> specification, Sorting<A> sorting, Range range);

    Stream<K> keys(Specification<A> specification, Sorting<A> sorting, Range range);
    
}
```

## FluentAssembler

### Allow to fail with custom exception

The `orFail()` method of the `FluentAssembler` DSL should has an overload allowing to specify the exception:

```java
public class SomeClass {
    @Inject
    private FluentAssembler fluentAssembler;
    
    public void someMethod() {
        fluentAssembler
            .merge(myDto)
            .into(MyAggregate.class)
            .fromRepository()
            .orFail(() -> throw new NotFoundException("MyAggregate #" + myDto.getId() + " not found");
    }
}
```
