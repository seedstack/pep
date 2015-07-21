
- PEP: 005
- Title: View enhancements
- Authors: 
 - Adrien LAUER <adrien.lauer@mpsa.com>
 - Pierre THIROUIN <pierre.thirouin@ext.mpsa.com>
- Status: draft

# Abstract

This PEP is about creating a component that will orchestrate the creation of a paginated or chunked view from the parameters to the result. Its responsabilities would be:

- To compute the range from pagination or chunk information,
- To create a finder context object holding the range and possibly other information,
- To inject the finder context in the finder before invoking the finder method,
- To compute the adequate view from the returned result.

# Motivation

Producing paginated or chunked views can be somewhat difficult today as there is no component to orchestrate the whole process. This results in various support questions and sometimes developers
just don't care and avoid pagination completely. More automation of this will help our users to produce better applications.

# Specification

## DSL

The idea is to introduce a ViewBuilder component that can be injected. It will provide a DSL to fluently build a whole paginated or chunked view.

Example:

    @Inject
    ViewBuilder viewBuilder;

    PaginatedView<KeyRepresentation> keyView = viewBuilder
        .page(1)
        .sized(10)
        .from(finder(KeyFinder.class).allKeys("*toto*", isApprox, isMissing))
        .build();

The statically imported `finder` method returns a proxy that will record the invocation to `allKeys` with all the parameters but not execute the method. The invocation itself will be done in the `build`
step after all finder context (computed range and so on) is injected into the finder.

## Finder context injection

A finder context needs to be injected in the finder before invoking the method actually doing the query. This can be done through `@Inject` but the injected implementation must take care of concurrency
(probably through a ThreadLocal implementation).
