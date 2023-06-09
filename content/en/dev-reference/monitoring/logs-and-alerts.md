---
title: "Monitoring logs and alerts"
lead: "Brief explanation of Slingr and use cases for it."
date: 2020-11-16T13:59:39+01:00
lastmod: 2020-11-16T13:59:39+01:00
draft: false
images: []
menu:
  docs:
    parent: "dev-reference"
toc: true
weight: 150
---

Logs of the app can be access in the app monitor in the section `Logs`. There you will be able to
see what's going on in your app, which is very important when you need to diagnose issues or there
is something odd in the app.

In the logs you will see logs generated by the system as well as logs done using the Javascript API
method like **********    fijarseeeeeeeeeee **********
 
Logs are sorted from the oldest to the newest. If you scroll down more logs will be loaded
automatically until no more logs are found.

One thing to keep in mind is that there is a limit on the size used by logs (see 
[Logs and alerts]({{site.baseurl}}/app-development-environment-logs-and-alerts.html)). 
Once the amount of logs reaches that size, logs will be rotated and older ones will be lost.

## Filtering logs

Logs can be filtered by log level (info, warn, error), time and a search string.

When filtering by a time period you should introduce a time duration in the next input box, like 
`5m` (last five minutes), `2h` (last two hours) or `3d` (last three days). If you need more
precision you can use a date range.

Search string will match all log items where the string is found. You can search by Regular expressions by typing `regex: REGEX`
replacing REGEX with the regex to filter logs. Example, with `regex: .*GET.*` you can filter those logs that are GET access.

## Show IP
 
There is a flag called `Show IP`. If it is set, the IP where the request comes from will be displayed
in the logs (if available as the event could have been generated internally).

## Show locations
 
There is a flag called `Show location`. If it is set, the location (city/state/country) where the request comes from will be displayed
in the logs (if available as the event could have been generated internally).

## External only

If this flag is set, only logs related to the interaction of the app with external systems will be
shown. For example REST API calls or endpoints interactions.

## UI only

If this flag is set, only logs of request coming from the UI will be shown.

## Getting more information

Some logs provide more information that is not displayed in the log line because it would be too
much and would be hard to read. In those cases you will see a small button with the label `More info`
that when you click it you can see more details.

For example in an endpoint event you can see the data of sent by the endpoint, and when there
is an error you will be able to see where in the code the error was thrown as well as the stack
trace, which is very useful information when you are debugging.

Finally, when jobs are executed there is a button with the label `Go to job` that will take you
to the job details.

## Context

When you filter logs, you will see a button called `Context` next to each log line. If you click
there you will see the logs before and after that log. This will help you to figure out the context
of the log without having to remove the filters you have set.

## Alerts

Alerts are based on logs. For example in production you might want to get an alert when an error
or a warning is logged.

You can do that in the app monitor or app builder in the section `Environment settings > Logs and alerts`.
[You can find more information here]({{site.baseurl}}/app-development-environment-logs-and-alerts.html).

## Download logs

It is possible to export app logs to a compressed JSON format file based on current filtering selection. The log entries 
file to export will be generated in a background task. Once the task has finished the download of the file will start 
automatically.
