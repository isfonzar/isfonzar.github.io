---
layout: post
title:  "Data Models"
date:   2020-05-14 12:40:31 +0200
categories: system-design
comment: true
---

Data Modeling is most likely the first problem one encounters when designing a piece of software and perhaps the one that can have the most repercussions as the development process goes on.

//TODO should I differentiate data model from database model here?

The database model is the logic layout on how the data will be stored and represented within a system. It serves as the blueprint that architects and the application developers will follow in all matters related to the data (storing, acessing, etc).

## Database Models

### Hierarchical database model

The first database model to be created was the Hierarchical database model. Developed by IBM in the 1960s, the data in this model are organized into a tree-like structure.

All the data is represented as a tree of records nested within other records.

There are some difficulties in using databases that implement this model, mainly the difficulty of setting "many-to-many" relationships, lack of support of "joins"

Due to its shortcomings, some models were created to solve the limitations that existed within this one, two of them were of great impact: the *relational model* and the *network model*.

### Network Model


### Relation Model


## References

https://www.techopedia.com/definition/6762/database-model
https://en.wikipedia.org/wiki/Hierarchical_database_model