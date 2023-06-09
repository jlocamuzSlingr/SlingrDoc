---
title: "Entities"
lead: "Explanation of what entities are, features they have, and how they are used."
date: 2020-11-16T13:59:39+01:00
lastmod: 2020-11-16T13:59:39+01:00
draft: false
images: []
menu:
  docs:
    parent: "dev-reference"
toc: true
weight: 22
---

Entities are the main concept in a Slingr app. They define the data structure as well as
the behavior of an app. Entities could be companies, people, tasks, projects, etc.

Once the entity is created, it is possible to create records under that entity. For example
if you have an entity called `companies`, each company under that entity will be a record.

An entity will define the following things:

- **Settings**: these are things like name, label, history logs, indexes, etc.
- **Fields**: fields define the structure of the entity. The structure of fields will
  impact the database and the UI.
- **Actions**: here it is possible to define the "behavior" of the entity.
- **Permissions**: define who has access to operations, actions and fields in the entity.

## Settings

### Label

This is a human-readable name of the entity. You can use spaces, special characters and
mix upper case and lower case letters.

The label will be used to reference the entity in the UI. Changing the label of an entity
doesn't involve any kind of data refactorings.

### Name

This is the internal name of the entity. It cannot hold special characters or spaces.

The name will be used for many things in the app:

- **Database**: records for the entity will be stored under the entity name.
- **REST API**: the automatic generated REST API uses the entity name for URLs.
- **Scripts**: scripts will have probably have references to the entity by the entity name.

This means that changing the name of an entity will impact all the above things.

Changes in the database will be handled automatically by the platform when pushing changes,
so you don't need to worry about them.

Changes in the REST API will be done automatically. However if there were external apps using
the REST API, those apps will need to be adjusted.

Finally, scripts using the entity name to reference the entity will need to be manually
updated. In the near feature we will provide some helps on these cases.

### Type

Defines how the entity should behave:

- `Data`: this means that the entity will holds app data. This is the most common case
  and the default option. This data won't be included in backups and won't be synced
  between environments and linked apps.
- `Enum`: these are entities that hold (usually) a small number of records that are in
  some way part of the metadata of the app, for example type of orders, states, etc.
  Records of this entity will be included in backups and synced to the different 
  environments and linked apps. You can define how this sync will happen:
  - `Not syncing`: this will make the entity behave as a data entity, where no records
    are synced.
  - `Create new records`: only new records will be synced, but no records will be deleted
    or updated in the target environment. For example if you created and updated records
    in the production environment, these changes will be kept.
  - `Create and update records`: new records will be synced and existing records will be
    updated. This means that if you made changes in one existing record in the target
    environment, sync will override those changes. Records created on the target environment
    won't be affected.
  - `Full sync`: when syncing the platform will make sure that records in the target
    environment are the same as the records in the source environment; in other words, any
    record created in the target environment will be deleted, and updated records will be
    overridden with the records from the source environment. On the other hand when pulling,  
    will add any non-existing record into the target environment and if it does exist,
    keeps the record from the environment which has the latest version.
- `Calculated`: this means that the entity will calculate its data. This is useful to model external data
   that has to be represented in our app. The behavior it's the same as for `Data` entities.
- `System`: this type of entities are created and managed by the platform and they can be extended by adding 
   custom configuration like entity fields and actions. The metadata autogenerated by the platform is immutable. 
   Entities of this type are located under the system folder `System`.
   An example of a system entity is `Users`. This entity is created when the app is provisioned. 
    
### Parent entity

An entity can inherit from another entity. This is helpful to avoid repeating things in the 
same app.

Extending an entity from another one means that fields, actions and indexes in the parent entity
will be available in the child entity.   

### Abstract

When you set a parent entity it is possible to flag an entity as abstract. When you do that it means 
that no instances of this entity will exist. So it won't be possible to create views for an abstract
entity.

### Record label

Records have a label that will be used as a summary of the record in different places of
the app. For example in relationship fields, when selecting a record, the label will be
displayed.

Usually the label should be representative of what the record holds. For example, if the
entity contains information about people, a good label could be the full name of the person.

There are two ways to define the label of records:
 
- `Field`: you have to select a field from the entity and it's string value will be used
  as the label of the record.
- `Script`: if you need more flexibility you can write a script that should return the
  label of the record.


{{< js_script_context context="recordLabelScript">}}


Keep in mind that changing the record label definition will trigger data refactorings to update label on
all records under the entity as well on record of entities that have relationship fields pointing to 
records on the entity where label was modified.

### Indexes

Indexes allow to improve the performance of queries. They follow the same concept of indexes
in traditional databases. For example if you have an entity with over a million records and you
make a lot of queries by the field `sku`, it makes sense to create an index for the field `sku`
to speed up those queries and avoid a full scan.

Indexes can be created directly in the `Indexes` section of an entity, or they will be created when
you flag a field as `Unique` or `Indexable`.

Each index has the following properties:

- `Name`: this is the name of the index in the database. It is calculated and read-only.
- `Type`: the type of index. This is calculated automatically based on the selected fields. Possible types
  are:
  - `Normal`: these are regular indexes.
  - `Relationship`: when the field is applied to a relationship, user, group or file field, two indexes will
    be created: one for the ID and another for the label, full name, or name (depending on the type).
  - `Compound`: when more than one field is selected, the type of the index will be compound.
- `Status`: this is the current status of the index as you can create it on the builder but it won't be created
  in the database until changes are pushed or synced. During the creation of the index there could be issues,
  and you can check the status to make sure it was created successfully.
- `Unique`: this flag indicates if the values for this index must be unique. If there are already records and
  there are duplicated values, the creation of the index will fail.
- `Fields`: these are the fields that will be part of the index. If you select more than one field keep in mind
  that in order to use the index in queries you need to query by all those fields.

Keep in mind that adding and removing indexes will cause indexes to be dropped or built, which might
take some time on entities with many records.

Also remember that indexes have an overhead during creation, update and delete of records as indexes
need to be updated. For this reason only create indexes when they are really needed.

### Global search

If you enable the flag `Allow global search` for an entity, an index will be created. This index will allow to
efficiently query for words in any field of the record. It is like searching something on any fields.

[You can find more information on how global search works here]({{<ref "/dev-reference/queries/query-language.md#global-search">}}).

### Record validations

Record validations allow to perform more complex validations across all fields in the record and 
using other services that are not available in fields' rules.

For example you might have an endpoint for an address validation service that you can use to 
validate addresses, or you just want to make sure that when one option is set in one field, 
the value of another field needs to match some specific pattern.

This is the script context:

{{< js_script_context context="recordValidationsScript">}}


### Lookup fields

Fields configured with this feature are going to be considered during record fetching. In order to lookup for 
specific record these fields are going to be used together with the default system fields: `ID` and `label`.  

Lookup fields require to have the `unique` flag enabled.

### Detailed logs
This feature is to enable more detailed logging for the following record operations: create, edit, delete and execution 
of actions over data records. 

This feature can be activated at entity level through this setting:
- `Enable detailed logging`: Whether enable detailed logging for record operations.

### History logs

This feature allow to log changes in records, which could be very useful for auditing purposes. For
example you can see when the record was created, the different changes made to the record over
time, as well as the fields modified and people involved in those changes.

When history logs are enabled for an entity you will be able to query logs of a record (or even
of all records at the same time) using the REST API or the UI. See 
[History]({{<ref "/dev-reference/rest-apis/apps-api.md#history">}}).

There are many settings you can control to decide what you want to log to the history of records:

- `Logs expiration`: allows to define rules to determine when logs have to expire.
  - `Delete policy`: indicates what to do with the history of a record when it is deleted.
    - `Delete history with record`: the history logs will be deleted when the record is deleted.
    - `Keep history`: the history logs of the record will be kept. When this option is selected
      you can specify more details about how long the history logs will be kept, which coud be
      a number of days/weeks/months.
- `User events`: user events are those changes made as a consequence of an external request, for
  example from the REST API or the UI. These are the events
  - `Ignore fields`: if you don't want to track changes on some specific fields when changes are 
    done through the REST API or the UI, you can put those fields in this list. This might be needed
    if you have fields that are updated all the time or if the content is too much ot track it.
  - `Record created`: if this flag is set, when a record is created from the REST API or the UI,
    a log for it will be created.
  - `Record changed`: if this flag is set, when a record is updated from the REST API or the UI,
    a log for it will be created.
  - `Record deleted`: if this flag is set, when a record is deleted from the REST API or the UI,
    a log for it will be created.
  - `Action executed`: if this flag is set, when an action is executed over an individual record
    from the REST API or the UI, a log for it will be created.
- `Events from scripts`: these are events generated from scripts in the app. For example when a
  script uses the method {% include js_symbol.html symbol='sys.data.save()' %}.
  - `Ignore fields`: if you don't want to track changes on some specific fields when changes are 
    done from a script in the app, you can put those fields in this list. This might be needed
    if you have fields that are updated all the time or if the content is too much ot track it.
  - `Record created`: if this flag is set, when a record is created from a script in the app,
    a log for it will be created.
  - `Record changed`: if this flag is set, when a record is updated from a script in the app,
    a log for it will be created.
  - `Record deleted`: if this flag is set, when a record is deleted from a script in the app,
    a log for it will be created.
  - `Action executed`: if this flag is set, when an action is executed over an individual record
    from a script in the app, a log for it will be created.
- `System events`: these are events that are a side effect of other operations. For example if you
  update the label of a record and it is referenced by another record, the relationship will be
  updated. Usually you don't want to track these kinds of changes.
  - `Cascade updates`: these events are generated when a change in another record needs to be
    propagated to other records. For example when you have copied fields in a relationship
    field and those fields are updated in a record, they need to be copied to relationship fields
    referencing that record.
  - `Refactorings`: if this flag is set, when changes are done in the entity definition that 
    produce refactorings on records, a log for it will be created. For example if you delete
    a field on the entity or change the label definition.

## Data generation settings

This settings are only available for `Calculated entities`.

### Schedule time expression

This expression is to determine when the script for generate data should run. The minimum frequency supported is 15 
minutes.  Please check this [documentation](http://www.quartz-scheduler.org/documentation/quartz-2.0.2/tutorials/tutorial-lesson-06.html) 
for more information on the format of the expression.

### Condition type

Allows to select if data should always be generated when time expression it's met or you can evaluate it by using a script.

### Condition

This script it's for determine if data should be re-generated or not and it's only available if you choose `Script` as condition type. It should return `true` if that's the case
or `false` otherwise. `Data generation script` will run only if this script return `true`.

### Data generation script

This script should return a list of either [Record objects]({{<ref "/dev-reference/scripting/sys-data.md#sys.data.Record">}}) or `JSONs` that can be mapped to a [Record object]({{<ref "/dev-reference/scripting/sys-data.md#sys.data.Record">}}).

### Behavior during regeneration

This flag allows to configure the behavior when data it's being generated. It apply to data queries.
If flag it's set to `true` then when trying to read any data for this entity, if data it's being generated at that moment that read
operation will wait until data generation ends.
If flag it's set to `false` then then when trying to read any data for this entity, if data it's being generated at that moment
an `AccessForbiddenException` will be throw.

## Fields

Fields define what information will be stored in records of the entity.

Slingr allows to have more complex structures than you usually can use in traditional 
relational database. For example it is allowed to have multi-valued fields as well as
nested fields. These features make it is easy to define a more natural model.

For example you might have an entity with this structure:

- `name`
- `type`
- `phoneNumbers` (multi-valued)
- `addresses` (multi-valued)
  - `addressLine`
  - `zipCode`
  - `state`
  
Each field has a type, which defines which data can be stored there as well as rules and
display options. [You can check the copied field types here]({{<ref "/dev-reference/field-types/overview.md">}}).
 
Apart from the type-specific settings, all fields share some common features. For more
details check [Fields]({{<ref "/dev-reference/data-model-and-logic/fields.md">}}).

There are two special field types that are worth mentioning:

- **Nested fields**: this type can hold other fields inside of it. This allows to have
  multiple levels of nesting. [You can find more information here]({{<ref "/dev-reference/field-types/special/special.md">}}).
- **Relationship**: this type allows to reference another record of any entity. This way
  it is possible to create relationships between data. [You can find more information
  here]({{<ref "/dev-reference/field-types/special/relationship.md">}}).

### Changes on fields and data refactorings

If you make changes to the structure of the entity by removing fields or making
changes to existing fields in a way that affect the data structure (for example you change
the field name), when those changes are pushed or synced, data refactorings will be done
to adjust all records to the new structure.

For example if you rename a field from `type` to `category`, when changes are pushed or
synced a refactoring over all records will be done to rename field `type` to `category`.

A more complex case happens when you change the type of the field. Let's suppose that the
field `itemCode` was an integer field but it is changed to a text field. In this case the
data refactoring will convert numbers to strings and the value will be preserved.

However, if you are converting in the other way around, the conversion might not be valid
in some cases. For example if the value was the string `"10"`, then it can be safely converted
to the number `10`, but if the value was `A10` then it is not a valid integer number and
the field value will be set to `null`.

The rule during conversions is that the original value will be converted to its string
representation, and the new type will try to parse that string representation. If the parsing
fails, the value will be set to `null`.

Finally, another conversion that can affect the structure is multiplicity changes. If a field
is changed from single-valued to multi-valued, existing value will be set as the first value
in the field. On the other hand, if the multiplicity is changed from multi-valued to single-valued,
only the first value will be kept, discarding any additional value of the field.

### Changes in rules

If there are changes in the rules of a field, it is possible that records that were valid before
are not longer valid. For example you have a text field and you add a maximum length of 10
characters. If there were already records with values longer than 10 characters for that field,
those records won't be valid any longer.

When this happens records will be kept as they are. If you try to update them the validation error
will show up and you will need to fix it before proceeding.

## Actions

Actions allow to define some behavior on records. For example, in an entity that holds tasks, you
could have an action to complete it, that will check that all pre-conditions are met, update the
status and notify people involved in the task.

There are basically two types of actions:

- `One record`: these actions are applied to one record at a time. Even when it is possible to 
  select many records through the UI or send many IDs on the REST API, this action will be applied 
  at one record at a time. The action doesn't know how many records are involved, it is only aware 
  of the record the action is being applied.
- `Many records`: these actions take a query as parameter that defines the selection of records. 
  This way the action knows all the records involved and it can do something with all of them at 
  the same time. For example you could have an action to send a summary of many tasks in one email, 
  which is not possible to do with actions that are executed over individual records.
  
For more information about actions, please check the documentation for 
[Actions]({{<ref "/dev-reference/data-model-and-logic/actions.md">}}).

## Record Listeners

Record Listeners can be hooked to different record events, like record created, changed, action executed, etc.

Those listeners will be listed in the entity, however they can also be managed from
the `Model > Listeners` section in the app builder.

For `Caculated entities` they have a different behavior. These listeners are used to update the records and they will be only listed in the calculated entity.

For more information check the documentation of [Listeners]({{<ref "/dev-reference/data-model-and-logic/listeners.md">}}).

## External Listeners

These listeners are only available for `Caculated entities`. External Listeners can be hooked to different record events, like record created, changed, action executed, etc.

Those listeners will be listed in the entity, however they can also be managed from
the `Model > Listeners` section in the app builder.

For more information check the documentation of  [Listeners]({{<ref "/dev-reference/data-model-and-logic/listeners.md">}}).

## Permissions

Permissions allow to define which operations can be done for each group on records of 
the entity.
  
Permissions for an entity can be handled right in the entity definition, but it is just
a different view of what you can configure in groups. It is oriented so you can easily
configure permissions on the entity for all existing groups.

When a new entity is added, no permissions are added to any group by default.

For more information about permissions please refer to [Groups]({{<ref "/dev-reference/security/groups.md">}}).
