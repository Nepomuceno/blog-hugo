---
title: Imutable data retention pipeline
tags:
---

## Retention goal

The idea it is really simple you want to refgister dome long term data based on a data retention pipeline, some of the requremns can make it a bit trickier you want it not to be updated what means that you want this data to be read only and at the same time you want to be able to query this data.

There are plenty of solutuions that would do that but I wanted somethin that not only could do that but that could do that and do it cheap and retain data for years at least 5.

## The basic architecture

The basic architecture fot thsi data retention it is really simple.

You start wiht a data ingestion queue. just some place where you could send this data easialy from many diferent data sources.

This data source needs to write to a readonly storage that will keep this data content.

The third mechanism would be a query engine that wouyld read this data index it and allow me to query it.

## Building blocks

To put togethert building blocks for that it is not that hard.

>Disclaimer here that I currently work for microsoft and directly with Azure

The cloud it is there to give you some option on how to do it.

to our data ingestion I will be using azure event hubs. It is a really secure data queue that scale really well and it is dirty cheap. can handle over 100.000 log events per day costing me [Insert here vent hubs price]()

For the data store I will be using azure data lake, azure data lake lets you restric the access to the data but more than that it also allows you to audit who access that data when. This will be really usefull when we talk later about how can we monitor and check who is actually reading this data.

To connect the data ingestion and the data lake we will just use the capture feature from event hubs. You need to follow some manual steps to connect this right now but this shpould become fairly easy on the comming days. You can check on my post [Using azure data lake for event hubs capture]() how I did automate that.

To to the index I will use auzre search for quick indexing the data and azure data lake for analytics and exploration queries on that data.

## Talk it is cheap show me the code

### 1. The generator

The code for the data generator irt is quite simple I will be simulating some scenatriosn that are fairly common.

I just wrote a basic .net core console app to generate this code (even after all those years and languages c# still being my language of choice).

```
```

The important part to notice it is that I have some fixed fields on my events `_s` (the event session) `_t` (the event type) those are fields that would be required in every log that I record from the application. Lets talk a little bit about event logging: the idea of creating a whole data lake for your event or logs it is to try to aggregate as much data about your system that you can. in order to do that you need to be flexible in your data schema, but at the same time you want to have some consistency between diferent entries in yuor system. the easiest way that I found so far to achieve that it is to use the namespace schema with version for event types.

Basically the idea it ia that all the events that you registry must have a type, and this type must have a namespace. the namespace should be managed on the coorporation level, to guarantee consistency, means that a new team will not come up with a namespace that it is currently in use by another team. a typical type for the event would include `{namespace}.{application}.{event_type}.{version}` I had some long discussion on should I put application before event_type or event type befor application I found out that you are mostly looking for information inside a given application or set of applications so I normally include it first. important to notive here that this is just one suggestion. the only golden rule here it is that events of the same time will follow the same schema ant that it is where the most important part of the event type it is the `{version}` in the end, the idea it is simple. Every time that you change how you capture a propertie you must change at least the version of the event.




I am now collecting data if you are trying to implement this in your organization I will always tell you to rush this part as fastas you can. you can always go back and fix other mistakes you can not go back in time and collect data that it is not there anymore.

