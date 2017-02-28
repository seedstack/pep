PEP: 006
Title: Business framework 4.0
Author: Adrien LAUER <adrien.lauer@mpsa.com>  
Status: draft  

# Abstract

The goal here is two-fold:
* Refactor business framework legacy parts into something more modern,
* And take advantages of Java 8 features everywhere it makes sense.

# Motivation

We want to improve the developer experience of using the business framework. For this we must fix long-standing oddities and use newer
features of the language where it makes sense. Functional upgrades of existing patterns can also be considered.

# Specification

## GenericRepository

The `GenericRepository` interface is a left-over from older versions and is replaced in purpose by the `Repository` interface. 
It should be removed.

## GenericFactory / Factory

To be consistent with other patterns, `GenericFactory` should be renamed `Factory` and `Factory` should be removed. The create(...)
method should have a default implementation in the interface so it won't need to be implemented by custom factories.

## Repository

TBD
