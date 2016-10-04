---
layout: post
title: Test and seed data in CUBA applications
description:
modified: 2016-10-04
tags: [cuba, test-data]
image:
  feature: seed-and-test-data-in-cuba-land/feature.gif
---

One thing that is kind of an evergreen in application development is the question about how to inject seed and / or test data into the system. In this blog post, we will have a look about how this topic is covered in CUBA-land and what we can possibly do to extend the functionality.

<!-- more -->

## General options for data import / export

When we look at what general options are availiable in a normal CUBA application at least the following list comes to my mind:

* JSON based im- / export of entity instances through the [entity inspector](https://doc.cuba-platform.com/manual-6.2/entity_inspector.html)
* SQL based export for entity instances through "System information" --> "Insert script"
* SQL / groovy based import through [DB scripts](https://doc.cuba-platform.com/manual-6.2/db_scripts.html) that are run in the init phase (like 30.create-db.sql)
* REST JSON im- / export for entity instances through the generic [REST API](http://files.cuba-platform.com/swagger/#/Entities)

There are [other valid options](http://stackoverflow.com/questions/22269307/inserting-initial-data-jpa) that aren't that integrated into the CUBA platform as well, but we will not cover these in details.

If i got it correctly, the JSON based im- / export has been added later to the platform and therefore can be seen as the newer version of the SQL based approach. So let's have where the differences are and what benefits each approach brings to the table.

| Feature | JSON based | SQL based |
|---------------------------|:-------------:|:-------------:|
| *automatic load at startup* | ☐ | ☑ |
| *distinction between seed data and test data* | ☑ | ☐ |
| *export multiple instances* | ☑ | ☐ |
| *association support*       | ☑ | ☑ |
| *DBMS independent*          | ☑ | ☐ |
| *API syntax*                | ☑ | ☐ |

As we see we both options have their advantages. But the main reason for using the SQL based approach is because there is already a mechanism to bootstrap this data with the idea of the [DB init scripts](https://doc.cuba-platform.com/manual-6.2/db_scripts.html).

The problem with this approach though is that you can't really use this feature if you want to distinct between data, that is necessary for production usage: [seed data](http://edgeguides.rubyonrails.org/active_record_migrations.html#migrations-and-seed-data) and data that is used for your interal test systems e.g. which might be part of the codebase, but should not leak into production. This is not always necessary, like when you distinct the test data from your application code and insert the data via a REST API e.g. after the application has started, but sometimes it can be handy to have the test data alongside with the code.

## Enable automatic JSON load at startup
Let's try to enhance the JSON based approach so that instead of having to manually import the data after the application started the JSON files will be picked up at application start just like the DB init scripts do.

For this, i created an example application: [cuba-example-json-testdata](https://github.com/mariodavid/cuba-example-json-testdata)
