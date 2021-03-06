---
layout: doc_page
title: "Querying"
---

<!--
  ~ Licensed to the Apache Software Foundation (ASF) under one
  ~ or more contributor license agreements.  See the NOTICE file
  ~ distributed with this work for additional information
  ~ regarding copyright ownership.  The ASF licenses this file
  ~ to you under the Apache License, Version 2.0 (the
  ~ "License"); you may not use this file except in compliance
  ~ with the License.  You may obtain a copy of the License at
  ~
  ~   http://www.apache.org/licenses/LICENSE-2.0
  ~
  ~ Unless required by applicable law or agreed to in writing,
  ~ software distributed under the License is distributed on an
  ~ "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  ~ KIND, either express or implied.  See the License for the
  ~ specific language governing permissions and limitations
  ~ under the License.
  -->

# Querying

Apache Druid (incubating) queries are made using an HTTP REST style request to queryable processes ([Broker](../design/broker.html),
[Historical](../design/historical.html). [Peons](../design/peons.html)) that are running stream ingestion tasks can also accept queries. The
query is expressed in JSON and each of these process types expose the same
REST query interface. For normal Druid operations, queries should be issued to the Broker processes. Queries can be posted
to the queryable processes like this -

 ```bash
 curl -X POST '<queryable_host>:<port>/druid/v2/?pretty' -H 'Content-Type:application/json' -H 'Accept:application/json' -d @<query_json_file>
 ```
 
Druid's native query language is JSON over HTTP, although many members of the community have contributed different 
[client libraries](../development/libraries.html) in other languages to query Druid. 

The Content-Type/Accept Headers can also take 'application/x-jackson-smile'.

 ```bash
 curl -X POST '<queryable_host>:<port>/druid/v2/?pretty' -H 'Content-Type:application/json' -H 'Accept:application/x-jackson-smile' -d @<query_json_file>
 ```

Note: If Accept header is not provided, it defaults to value of 'Content-Type' header.

Druid's native query is relatively low level, mapping closely to how computations are performed internally. Druid queries 
are designed to be lightweight and complete very quickly. This means that for more complex analysis, or to build 
more complex visualizations, multiple Druid queries may be required.

## Available Queries

Druid has numerous query types for various use cases. Queries are composed of various JSON properties and Druid has different types of queries for different use cases. The documentation for the various query types describe all the JSON properties that can be set.

### Aggregation Queries

* [Timeseries](../querying/timeseriesquery.html)
* [TopN](../querying/topnquery.html)
* [GroupBy](../querying/groupbyquery.html)

### Metadata Queries

* [Time Boundary](../querying/timeboundaryquery.html)
* [Segment Metadata](../querying/segmentmetadataquery.html)
* [Datasource Metadata](../querying/datasourcemetadataquery.html)

### Search Queries

* [Search](../querying/searchquery.html)

## Which Query Should I Use?

Where possible, we recommend using [Timeseries]() and [TopN]() queries instead of [GroupBy](). GroupBy is the most flexible Druid query, but also has the poorest performance.
 Timeseries are significantly faster than groupBy queries for aggregations that don't require grouping over dimensions. For grouping and sorting over a single dimension,
 topN queries are much more optimized than groupBys.

## Query Cancellation

Queries can be cancelled explicitly using their unique identifier.  If the
query identifier is set at the time of query, or is otherwise known, the following
endpoint can be used on the Broker or Router to cancel the query.

```sh
DELETE /druid/v2/{queryId}
```

For example, if the query ID is `abc123`, the query can be cancelled as follows:

```sh
curl -X DELETE "http://host:port/druid/v2/abc123"
```

## Query Errors

If a query fails, you will get an HTTP 500 response containing a JSON object with the following structure:

```json
{
  "error" : "Query timeout",
  "errorMessage" : "Timeout waiting for task.",
  "errorClass" : "java.util.concurrent.TimeoutException",
  "host" : "druid1.example.com:8083"
}
```

The fields in the response are:

|field|description|
|-----|-----------|
|error|A well-defined error code (see below).|
|errorMessage|A free-form message with more information about the error. May be null.|
|errorClass|The class of the exception that caused this error. May be null.|
|host|The host on which this error occurred. May be null.|

Possible codes for the *error* field include:

|code|description|
|----|-----------|
|`Query timeout`|The query timed out.|
|`Query interrupted`|The query was interrupted, possibly due to JVM shutdown.|
|`Query cancelled`|The query was cancelled through the query cancellation API.|
|`Resource limit exceeded`|The query exceeded a configured resource limit (e.g. groupBy maxResults).|
|`Unauthorized request.`|The query was denied due to security policy. Either the user was not recognized, or the user was recognized but does not have access to the requested resource.|
|`Unknown exception`|Some other exception occurred. Check errorMessage and errorClass for details, although keep in mind that the contents of those fields are free-form and may change from release to release.|
