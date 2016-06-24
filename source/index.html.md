---
title: API Reference

language_tabs:
  - python

toc_footers:
  - <a href='http://www.vertical-knowledge.com'>Vertical Knowledge</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Vertical Knowledge data API.  This is an early developer release of the API and should not
be used in production environments.  [Issues](https://github.com/vkapi/idyia/issues) can be submitted for any questions or assistance.  For API
access please contact our [team](mailto:info@vertical-knowledge.com).

This is a RESTful JSON API and helps connect users with VK's data.  Data is separated by several tiers of structure:

- Project:  The system or source of data.
- Table:  A specific data model or 'database table' of data in a project.  Data is delivered flat but may have implied relations to other tables in a project.  
- Batch:  A snapshot of data in time based on when the crawlers ran.

Typical use cases well have users logging in, searching through their projects, choosing a table, and identifying which
batch they wish to review.  Once this information is collected, the search endpoint can be consumed to process the actual data.  

Schemas for data sets can be provided by GETing a specific project.  

# Basic API information

Most resources return a Meta object and an objects list.  The Meta object helps the user know results, next pages,
and other information about their request.  The objects list contains the actual data itself.

```json
  {
    "meta":
    {"limit": 20,
     "next": null,
     "offset": 0,
     "previous": null,
     "total_count": 1
   },
   "objects": [...]
  }
```


### Meta Attributes

Attribute | Description
--------- | -----------
limit | The amount of objects to return.  Defaults to 10, max limit is 1,000
next | Provides a suggested next 'page' link based on your limit
previous | Provides a suggested previous 'page' link based on your limit
offset | Amount of records to skip before returning results.  Used for pagination
total_count | Total amount of records available with your query


### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
limit | 10 | The amount of objects to return.  Defaults to 10, max limit is 1,000
offset | 0 | Amount of records to skip before returning results.  Used for pagination


# Authentication

> Authorization requires a previously provided username and password:


```python
import json
import requests

api_url = "http://idyia.vk.ai/search_api/v1"

data = json.dumps({"username": "USERNAME", "password": "PASSWORD"})
res = requests.post("{0}/login/".format(api_url), data=data)
session = json.loads(res.content)['session']
```

> returns:

```json
[
  {
    "session": "SESSION_KEY",
    "session_set": "2016-06-24T14:21:11.136996"
  }
]
```

> Make sure to replace `USERNAME` and `PASSWORD` with your credentials.

All resources other than login require a session key.  Session keys are received after successfully POSTing
to the login resource.  A session will expire after 15 minutes of inactivity.  


<aside class="notice">
Be sure to add session=SESSION to all other requests.  You will receive a 401 if your session is invalid or expires.
Additional logins will cancel out previous sessions.  
</aside>

# Logout

> Logging out requires an active session key:


```python
params = {"session": session}
res = requests.delete("{0}/logout/".format(api_url), params=params)
```


> returns a 204 if successful.

Logging out instantly terminates a session and will prevent any other requests
until the user logs in again.

# Project

## Get ALL projects


```python
params = {"session": session}
res = requests.get("{0}/project/".format(api_url), params=params)
```

> The above command returns JSON structured like this:

```json
  {
    "meta":
    {"limit": 20,
     "next": null,
     "offset": 0,
     "previous": null,
     "total_count": 1
   },
   "objects": [
     {
       "guid": "351b6460bcdf250e62e1c618a4dca27891efdd78c44725eac40784488b553e1e",
       "id": 1,
       "name": "Bestsellers",
       "pre_approve": true,
       "resource_uri": "/search_api/v1/project/1/"
     }
   ]
  }
```

This endpoint retrieves all projects your credentials have access too.


### HTTP Request

`GET http://idyia.vk.ai/search_api/v1/project/?format=json&session=SESSION_KEY`

### Project Attributes

Attribute | Description
--------- | -----------
guid | Unique text identifier for the project
id | Unique identifier for the project
name | Project name
pre_approve | Determines if batches are automatically validated or if the must be approved by VK first
resource_uri | Resource path to directly return the object


## Get a specific Project


```python
params = {"session": session}
res = requests.get("{0}/project/1/".format(api_url), params=params)
```


> The above command returns JSON structured like this:

```json
  {
    "guid": "351b6460bcdf250e62e1c618a4dca27891efdd78c44725eac40784488b553e1e",
    "id": 1,
    "name": "Bestsellers",
    "pre_approve": true,
    "resource_uri": "/search_api/v1/project/1/",
    "schema": {
      "tables": [
        {"fields": [
          {"data_type": "str", "name": "name"},
          {"data_type": "str", "name": "brand"},
          {"data_type": "datetime", "name": "created_at"},
          {"data_type": "float", "name": "rating_count"},
          ...
         ],
         "name": "Item"
        }
      ]
    }
  }
```

This endpoint retrieves a specific project.  When retrieving a specific project the schema is provided.


### HTTP Request

`GET http://idyia.vk.ai/search_api/v1/project/<ID>/?format=json&session=SESSION_KEY`

### Project Attributes

Attribute | Description
--------- | -----------
guid | Unique text identifier for the project
id | Unique identifier for the project
name | Project name
pre_approve | Determines if batches are automatically validated or if the must be approved by VK first
resource_uri | Resource path to directly return the object
schema.tables | List of tables associated with the project
schema.tables.name | Table name
schema.tables.fields | List of fields associated with a table
schema.tables.fields.name | Field name
schema.tables.fields.data_type | Primitive of the field.  Typically string, datetime, integer, or float


# Table

## Get ALL Tables


```python
params = {"session": session}
res = requests.get("{0}/table/".format(api_url), params=params)
```

> The above command returns JSON structured like this:

```json
  {
    "meta": {
      "limit": 20,
      "next": null,
      "offset": 0,
      "previous": null,
      "total_count": 1
    },
    "objects": [
      {
        "id": 1,
        "name": "Item",
        "resource_uri": "/search_api/v1/table/1/"
      }
    ]
  }
```

This endpoint retrieves all tables your credentials have access too.


### HTTP Request

`GET http://idyia.vk.ai/search_api/v1/table/?format=json&session=SESSION_KEY`

### Table Attributes

Attribute | Description
--------- | -----------
id | Unique identifier for the table
name | Table name
resource_uri | Resource path to directly return the object


## Get a specific Table


```python
params = {"session": session}
res = requests.get("{0}/table/1/".format(api_url), params=params)
```


> The above command returns JSON structured like this:

```json
  {
    "id": 1,
    "name": "Item",
    "resource_uri": "/search_api/v1/table/1/"
  }
```

This endpoint retrieves a specific table.


### HTTP Request

`GET http://idyia.vk.ai/search_api/v1/table/<ID>/?format=json&session=SESSION_KEY`

### Table Attributes

Attribute | Description
--------- | -----------
id | Unique identifier for the table
name | Table name
resource_uri | Resource path to directly return the object


# Field

## Get ALL Fields


```python
params = {"session": session}
res = requests.get("{0}/field/".format(api_url), params=params)
```

> The above command returns JSON structured like this:

```json
  {
    "meta": {
      "limit": 20,
      "next": null,
      "offset": 0,
      "previous": null,
      "total_count": 1
    },
    "objects": [
      {
        "data_type": "str",
        "id": 23,
        "name": "name",
        "resource_uri": "/search_api/v1/field/23/"
      }
    ]
  }
```

This endpoint retrieves all fields your credentials have access too.


### HTTP Request

`GET http://idyia.vk.ai/search_api/v1/field/?format=json&session=SESSION_KEY`

### Field Attributes

Attribute | Description
--------- | -----------
id | Unique identifier for the field
data_type | Primitive of the field.  Typically string, datetime, integer, or float
name | Field name
resource_uri | Resource path to directly return the object


## Get a specific Field


```python
params = {"session": session}
res = requests.get("{0}/table/1/".format(api_url), params=params)
```


> The above command returns JSON structured like this:

```json
  {
    "data_type": "str",
    "id": 23,
    "name": "name",
    "resource_uri": "/search_api/v1/field/23/"
  }
```

This endpoint retrieves a specific field.


### HTTP Request

`GET http://idyia.vk.ai/search_api/v1/field/<ID>/?format=json&session=SESSION_KEY`

### Field Attributes

Attribute | Description
--------- | -----------
id | Unique identifier for the field
data_type | Primitive of the field.  Typically string, datetime, integer, or float
name | Field name
resource_uri | Resource path to directly return the object


# Batch

## Get ALL Batches


```python
params = {"session": session}
res = requests.get("{0}/batch/".format(api_url), params=params)
```

> The above command returns JSON structured like this:

```json
  {
    "meta": {
      "limit": 20,
      "next": null,
      "offset": 0,
      "previous": null,
      "total_count": 1
    },
    "objects": [
      {
        "created": "2016-06-22T18:23:18.017486",
        "external_date": "2016-06-09T06:18:15",
        "external_id": 17757,
        "id": 97,
        "resource_uri": "/search_api/v1/batch/97/",
        "status": "VERIFIED"
      },
    ]
  }
```

This endpoint retrieves all batches your credentials have access too.


### HTTP Request

`GET http://idyia.vk.ai/search_api/v1/batch/?format=json&session=SESSION_KEY`

### Batch Attributes

Attribute | Description
--------- | -----------
id | Unique identifier for the batch
created | When the batch was ingested into the API
external_date | Date of when the batch was collected
external_id | Batch ID in the collection system
status | If the batch is VERIFIED, STAGED (needs QA), NEW, INPROGRESS, or REJECTED
resource_uri | Resource path to directly return the object


## Get a specific Batch


```python
params = {"session": session}
res = requests.get("{0}/batch/1/".format(api_url), params=params)
```


> The above command returns JSON structured like this:

```json
  {
    "created": "2016-06-22T18:23:18.017486",
    "external_date": "2016-06-09T06:18:15",
    "external_id": 17757,
    "id": 97,
    "resource_uri": "/search_api/v1/batch/97/",
    "status": "VERIFIED"
  }
```

This endpoint retrieves a specific batch.


### HTTP Request

`GET http://idyia.vk.ai/search_api/v1/batch/<ID>/?format=json&session=SESSION_KEY`

### Batch Attributes

Attribute | Description
--------- | -----------
id | Unique identifier for the batch
created | When the batch was ingested into the API
external_date | Date of when the batch was collected
external_id | Batch ID in the collection system
status | If the batch is VERIFIED, STAGED (needs QA), NEW, INPROGRESS, or REJECTED
resource_uri | Resource path to directly return the object



# Search

## Retrieves data from a project, batch, and a table.  Requires all three paramaters.


```python
params = {"session": session, "project": 1, "batch": 97, "table": 1}
res = requests.get("{0}/batch/".format(api_url), params=params)
```

> The above command returns JSON structured like this:

```json
  {
    "meta": {
      "batch_external_date": "2016-06-09T06:18:15",
      "batch_external_id": 17757,
      "batch_status": "Verified",
      "limit": 10,
      "next": "/search_api/v1/search/?table=1&session=SESSION&project=1&batch=97&offset=10",
      "offset": 0,
      "pages": 3532,
      "previous": null,
      "total_count": 35318
    },
    "objects": [
      {
        "batch_id": 97,
        "data": {
          "brand": "MX",
          "category": "Baño",
          "created_at": "2016-06-09T06:29:37",
          "days_in_top_100": null,
          "is_available": 1,
          "marketplace_count": 6,
          "marketplace_price_max": null,
          "marketplace_price_min": null,
          "marketplace_price_raw": "EUR 3,04",
          "name": "Shower Shop UK - Anillas de metal par...",
          "price": 3.04,
          "price_max": null,
          "price_min": null,
          "price_raw": "EUR 3,04",
          "rank": 57,
          "rating_count": 3.0,
          "rating_value": 4.5,
          "sale_price": null,
          "sale_price_max": null,
          "sale_price_min": null,
          "sale_price_raw": null,
          "site_url": "http://www.amazon.es/item/dp/B005VDVH0I/"
        },
        "ds_id": "aa3fc703bee88332edbc1afebbffa588969700a1de0f0f2518850afb2db8113d",
        "index_date": "2016-06-23T00:51:09"
      }
      ...
    ]
  }
```

This endpoint retrieves all data your credentials and parameters have access too.


### HTTP Request

`GET http://idyia.vk.ai/search_api/v1/search/?format=json&session=SESSION_KEY&project=<PROJECT>&table=<TABLE>&batch=<BATCH>`

### Query Parameters

Parameter | Description
--------- | -----------
project | Unique identifier for the project you are searching
batch | Unique identifier for the batch you are searching
table | Unique identifier for the table you are searching

### Meta Attributes

Attribute | Description
--------- | -----------
batch_external_date | Date of when the batch was collected
batch_external_id | Batch ID from the collection system
batch_status | Date of when the batch was collected
external_id | Batch ID in the collection system
status | If the batch is VERIFIED, STAGED (needs QA), NEW, INPROGRESS, or REJECTED
pages | Estimated total pages based on limit and total_count

### Object Attributes

Attribute | Description
--------- | -----------
batch_id | Internal id of the batch
ds_id | Internal unique identifier of the data record
index_date | Date record was ingested into the API
data | Actual data for the object.  Schema based on project and table
