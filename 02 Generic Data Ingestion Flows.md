# Generic Data Ingestion Flows

Let us understand how to build Generic Data Ingestion flows in this session. 

* Recap of simple pipeline
* Recap of Flowfiles and Attributes
* Overview of Scheduling
* Files to Flowfiles - Options
* Build Generic Pipeline
* Quick Overview of NiFi Expression Language

The content will be free, however if you want to watch videos to understand key concepts feel free to [join our YouTube Channel as a member](https://www.youtube.com/channel/UCakdSIPsJqiOLqylgoYmwQg/join).

## Recap of simple pipeline

## Recap of Flowfiles and Attributes

## Overview of Scheduling

## Files to Flowfiles - Options

## Build Generic Pipeline

Here are the instructions building generic pipeline.
* Use listFile to get the files recursively.
* Configure Age Attributes as per requirement.
* Use fetchFile to fetch files.
* Update attribute to define target location using Update Attribute Processor. We can use NiFi Expression Language for the same.
```
${absolute.path:substringBeforeLast('/'):substringAfterLast('/')}
```

## Quick Overview of NiFi Expression Language

Let us understand more about NiFi Expression Language.
* It is extensively used to mutate flowfiles or attributes.
* We can also use NiFi Expression Language as part of conditional flows using processors such as **Route On Attribute**.
* NiFi Expression Language provides robust set of functions for data manipulation as well as traversing to JSON Paths.
* We have used expression language as part of previous topics in this module. We will see more as we get into more complex modules.
