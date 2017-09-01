# Object-Relational Mapping

## Table of Contents
1. [Introduction](#introduction)
2. [Repositories](#repositories)
3. [Data Mappers](#data-mappers)
4. [Units of Work](#units-of-work)

<h2 id="introduction">Introduction</h2>

Opulence utilizes the **repository pattern** to encapsulate data retrieval from storage.  [**Repositories**](orm-repositories) have [**data mappers**](orm-data-mappers), which directly interact with storage, eg cache and/or a relational database.  Repositories use [**units of work**](orm-units-of-work), which act as transactions across multiple repositories.  Unlike other popular PHP frameworks, Opulence does not force you to extend ORM classes in order to make them storable.  You can use POPOs (plain-old PHP objects).

<h2 id="repositories">Repositories</h2>

[**Repositories**](orm-repositories) act as collections of entities.  They include methods for adding, deleting, and retrieving entities.  The actual retrieval from storage is done through **data mappers** contained in the repository.  Note that there are no methods like `update()` or `save()`.  These actions take place in the data mapper and are scheduled by the **unit of work** contained in the repository.

<h2 id="data-mappers">Data Mappers</h2>

[**Data mappers**](orm-data-mappers) act as the go-between for repositories and storage.  By abstracting this interaction away from repositories, you can swap your method of storage without affecting the repositories' interfaces.

<h2 id="units-of-work">Units of Work</h2>

[**Units of work**](orm-units-of-work) act as transactions across multiple repositories, giving you "all-or-nothing" functionality.  They schedule updates, insertions, and deletions in the data mappers.  Because writes are all committed at once rather than throughout the lifetime of the request, the database connection is open for less time, giving you better performance.