---
title: "Bar chart widget"
lead: "Detailed explanation of settings available for bar chart widget views."
date: 2020-11-16T13:59:39+01:00
lastmod: 2020-11-16T13:59:39+01:00
draft: false
images: []
menu:
  docs:
    parent: "dev-reference"
toc: true
---

A bar chart provides a way of showing data values represented as vertical bars. It is sometimes 
used to show trend data, and the comparison of multiple data sets side by side.

## Entity

This is the entity the view will point to. Only records of this entity will be listed
in the bar chart widget.

## Label

This is a human-readable name of the view. You can use spaces, special characters and
mix upper case and lower case letters.

This label will be displayed at the top of the table widget view, so make sure you use something
users will understand.

## Name

This is the internal name of the view. It cannot hold special characters or spaces.

Usually you will use the name of the view in scripts and the REST API, so changing it
might affect your app and you will need to make some manual adjustments.

## Description

This is a human-readable description of the widget. You can use spaces, special characters and
mix upper case and lower case letters. You can internationalize it.

This description will be displayed in top of added widget in your dashboard.

## Allow Refresh

This is to Show/Hide refresh button. We can perform this action in order to update this widget 
information. Is true by default.

## List settings

### Filtering settings

These settings indicate how the listing should behave.

#### Sort field

This is the default sorting field for the listing.

Users might be able to change the default sorting if that's enabled in any of the columns.

#### Sort type

Indicates the direction of the sorting. 

Users might be able to change the default sorting if that's enabled in any of the columns.

#### Filter type

Can be expression or script. In case to select expression should set record filters. In other case should return
the script.

#### Record filters

Defines which records will be listed. Only records matching the given expression will be listed
in the table widget view.

#### Script

Return query parameter or queryMap. The query map object used to filter records. 
Check the [Query language]({{site.baseurl}}/app-development-query-language.html) documentation for the query map version.

#### Size

The maximum number of records to fetch when the view is loaded and when clicking in `More` to fetch
more records.


## Horizontal axis settings

### View type

Represent the field type to be used to generate the horizontal axis values. It can be an entity field or a
calculated field.

### Entity field

Allows to select an existing entity field. The value will be stored as a reference to metadata, being refactored in 
the same way when changes happen. The default value will be set only when creating a new record, which means it
won't have any effect when editing an existing record.

### Calculated field

The value of this kind of field is generated by script for horizontal axis.

### Horizontal bar

The configuration options for the horizontal bar chart are the same as for the bar chart. However, 
any options specified on the horizontal axis in a bar chart, are applied to the vertical axis in a horizontal bar chart.

### Grid lines

If false, do not display grid lines for this axis.

## Vertical axis settings

### Label

This is a human-readable name to be displayed in vertical axis. You can use spaces, special characters and
mix upper case and lower case letters.

### Ticks suggested min / max

The suggestedMax and suggestedMin settings only change the data values that are used to scale the axis. 
These are useful for extending the range of the axis while maintaining the auto fit behaviour.

### Grid lines

If false, do not display grid lines for this axis.

## Series

You can add series base on entity field or a calculated series. 

### Entity field

Allows to select an existing entity field. The value will be stored as a reference to metadata, being refactored in 
the same way when changes happen. The default value will be set only when creating a new record, which means it
won't have any effect when editing an existing record.

### Calculated field

The value of this kind of field is generated by script for vertical axis.

### Bar styling settings

#### Settings Mode

It can be simple or advanced. In case of simple mode just need set bar color and other settings are calculated or 
set by default. For advanced the developer should define each value.

#### Background color

This is the line color. It should be hexadecimal color code.

#### Border color
 
The bar border color. It should be hexadecimal color code.

#### Border skipped

The edge to skip when drawing bar.

#### Border Width

The bar border width (in pixels).

#### Hover background color

The bar background color when hovered.

#### Hover border width

The bar border width when hovered (in pixels).

## UI Message

Widgets can react to UI message. The common case is when need refresh the data or apply filters.

### Refresh and filter

It is possible send a message to refresh the widget using its default settings. This messages are sending to dashboard 
container and propagated to each widget. 

Additionally you can send a filter parameter in order to be applied to the query used to get the data. This query can be 
a [query]({{site.baseurl}}/app-development-js-api-data.html#sys.data.Query) object or a query map. In case to use a query 
object the filter just is applied if the entity is same to the entity set in the widget. Check the 
[Query language]({{site.baseurl}}/app-development-query-language.html) for more infomation.  

In order to apply filters is necessary set the `widgetContainer` with a list of objects in witch each one describes the 
container `name` and the `filter` to be applied on each widget. Additionally you can send `title` to update the widget 
title.

#### Example

In following example you can refresh and apply a filter sending the filter parameter to given widget container. 

```javascript
var query = sys.data.createQuery('salesInfoPeriod');
query.field('department').equals('Department A');

sys.ui.sendMessage({
  scope: 'view',
  name: 'refresh',
  views: ['salesDashboard'],
  widgetContainers: [
    {
        name: 'salesPerDepartment',
        filter: query
    }
  ]   
});
``` 
