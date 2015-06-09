# Object-Relational Mapping

## Table of Contents
1. [Introduction](#introduction)
2. [Repositories](#repositories)
3. [Data Mappers](#data-mappers)

<h2 id="introduction">Introduction</h2>
RDev utilizes the *repository pattern* to encapsulate data retrieval from storage.  *Repositories* have [*data mappers*](orm-data-mappers) which actually interact directly with storage, eg cache and/or a relational database.  Repositories use [*units of work*](orm-units-of-work), which act as transactions across multiple repositories.  Unlike other popular PHP frameworks, RDev does not force you to extend ORM classes in order to make them storable.  The only interface they must implement is `RDev\ORM\IEntity`, which simply requires `getId()` and `setId()`.

<h2 id="repositories">Repositories</h2>
*Repositories* act as collections of entities.  They include methods for adding, deleting, and retrieving entities.  The actual retrieval from storage is done through *data mappers* contained in the repository.  Note that there are no methods like `update()` or `save()`.  These actions take place in the *data mapper* and are scheduled by the *unit of work* contained by the repository.  [Read here](orm-data-mappers) for more information on data mappers or [here](orm-units-of-work) for more information on units of work.

> **Note:** In `get*()` repository methods, do not call the data mapper directly.  Instead, call `getFromDataMapper()`, which will handle managing entities in the unit of work.

<h2 id="data-mappers">Data Mappers</h2>
[*Data mappers*](orm-data-mappers) act as the go-between for repositories and storage.  By abstracting this interaction away from repositories, you can swap your method of storage without affecting the repositories' interfaces.