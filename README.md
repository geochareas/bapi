---
description: 👋 Welcome to SoftOne's Developer HUB
---

# SoftOne Developers Hub

**Overview**

Soft1 Services is a big and complex API that can be confusing for developers, especially those used to standard REST APIs. Unlike typical REST APIs, Soft1 Services is a Restless API, which makes it a bit challenging for new developers. That's where the SoftOne Developers Hub comes in – it's a user-friendly interface built specifically for Soft1 Services. It follows modern REST API standards and supports basic HTTP methods (GET, POST, PUT, DELETE) and JWT Authentication. The hub also simplifies things by providing clear response status codes to show how operations went. It handles the complexities of Soft1 services, like dealing with date and time and applying filters.&#x20;

Created by developers, for developers, the SoftOne Developers Hub aims to make working with Soft1 Services easier for everyone.

**Table of Contents**

* Overview
* Getting Started
  * Authentication
* Requests
  * API Base URL
  * Example Endpoints
  * Request Body
* Filters



### Getting Started

* Log in to [SoftOne Developer Hub](https://developers-stage.softone.gr/login?from=/requestAccess)
* Create a [new application](https://developers-stage.softone.gr/viewApps)

**Authentication**

1. Log in to your SoftOne installation and locate your serial number.
   * The serial number can be found in the upper right corner under User/Εγκατάσταση/Ταυτότητα/Serial No.
2. At the developers' Hub, [request access](https://developers-stage.softone.gr/requestAccess) to link a newly created App with an existing SoftOne serial number.
3. Approve the access request at SoftOne.
4. Navigate to the Request Access tab on developers' Hub to view all access requests. Copy the JWT Token necessary for API access.
5. Ensure the JWT Token is included in the Authorization header for HTTP requests.

### Requests

**API Base URL**

`api/v1/{entity}`

A comprehensive list of entities, along with their request/response sample bodies, can be found in the [web documentation](https://developers.softone.gr/docs).

For POST & PUT requests requiring a request body, the web documentation provides a sample request body containing only the essential fields necessary for a successful API call.

**Example Endpoints**

| HTTP Verb | Request URL        | Description                               | Response Status Code                                         |
| --------- | ------------------ | ----------------------------------------- | ------------------------------------------------------------ |
| GET       | api/v1/Sales       | Read (i.e. list) a resource collection    | `200-OK`, `204-No Content`                                   |
| GET       | api/v1/Sales/{key} | Read the resource                         | `200-OK`, `404-Not Found`                                    |
| POST      | api/v1/Sales       | Create new resource (ID set by service)   | `201-Created` with URL of created resource (Location Header) |
| PUT       | api/v1/Sales/{key} | Modify the resource with JSON Merge Patch | `200-OK`                                                     |
| DELETE    | api/v1/Sales/{key} | Remove the resource                       | `204-No Content`                                             |

**Request Body**

All request bodies must contain a "data" property encapsulating entity-specific fields. An optional "select" property may also be included to specify fields for the response body, akin to a SELECT statement in SQL.

**Example Request Body:**

When creating an entity which contains tables like items (ITELINES), we may want to set several quantities. In order to set `ITELINES.QTY1`, on the documentation we can see that we have to set the `Quantity_2` property

| quantity | <p>string or null</p><p>Set <code>Quantity</code> for ITELINES.QTY,<br><code>Quantity_2</code> for ITELINES.QTY1,<br><code>Quantity_3</code> for ITELINES.QTY2</p> |
| -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |

```json
{
    "data": {
        "transactionCode": 7021,
        "customerId": "32",
        "items": [
            {
                "itemID": 1191,
                "quantity_2": 1,
                "vatId": 0
            }
        ]
    },
    "select": "sales.documentCode,Sales.createdat,customerid,items.quantity_2"
}
```



### Query Operations

| Parameter name | Type         | Description                                                                  |
| -------------- | ------------ | ---------------------------------------------------------------------------- |
| `filter`       | string       | an expression on the resource type that selects the resources to be returned |
| `orderby`      | string       | a list of expressions that specify the order of the returned resources       |
| `skip`         | integer      | an offset into the collection of the first resource to be returned           |
| `maxpagesize`  | integer      | the maximum number of resources to include in a single response              |
| `select`       | string array | a list of field names to be returned for each resource                       |

### Filters

GET requests support filtering via the "filter" query parameter, which can be combined with various filtering operators.

| Operator                 | Description           | Example                       |
| ------------------------ | --------------------- | ----------------------------- |
| **Comparison Operators** |                       |                               |
| eq                       | Equal                 | city eq 'Redmond'             |
| ge                       | Greater than or equal | price ge 10                   |
| le                       | Less than or equal    | price le 100                  |
| **Logical Operators**    |                       |                               |
| and                      | Logical and           | price le 200 and price gt 3.5 |

**Date fields filtering :**

* The date format (YYYY-MM-DD) is supported for date filtering.
* The time portion of DateTime fields is unnecessary and can be omitted as it is internally managed.

Example: All sale documents of customer with VAT `000000000`

`GET https://developers.softone.gr/api/v1/sales?filter=customerVatCode eq '000000000'`

Example: All sale documents created at 03/01/2024 (date filter)&#x20;

`GET https://developers.softone.gr/api/v1/sales?filter=createdAt eq 2024-01-03`

Example: All sale documents with transaction codes `7001`,`7021,7061`

`GET https://developers.softone.gr/api/v1/sales?filter=transactionCode in (7001,7021,7041)`

Example: All items with retail price (`ITEM.RICER`) greater than 5 and less than 10 (combined filters)

`GET https://developers.softone.gr/api/v1/item?filter=retailPrice ge 5 and retailPrice le 10`

#### Order by

In order to sort the results of a list operation, you have to fill the parameters below

| Param       | Description                                                                | Default value |
| ----------- | -------------------------------------------------------------------------- | ------------- |
| orderby     | Field to sort with, should include the `asc` (default) or `desc` direction |               |
| maxpagesize | Maximum number of resources to include in a single page response           | 25            |
| skip        |                                                                            | 0             |

On the first request, you have to define the field and the direction you want to sort with. `maxpagesize` defaults to `25`, so the response with include a maximum of 25 results.

`GET https://developers.softone.gr/api/v1/sales?orderby=createdAt`

If your query contains more than one page, the response will contain a `lastKey` property.

```json
{
    "lastKey": "X0A002851C399B67F",
    "data": [
        ...
    ]
}
```

Use the `lastKey` query param and `skip` the current page

`GET https://developers.softone.gr/api/v1/sales?&orderby=transactioncode desc&lastKey=X0A002851C399B67F&skip=25`

On the next request, skip 50, 75, 100 etc and repeat until the response has no `lastKey` property (there are no pages left)

#### Select

Example: Select only the `documentCode`,`customerVatCode`,`customerId`,`createdAt`fields

{% hint style="info" %}
If the field does not contain the entity (e.g. sales), it will be automatically prefixed.
{% endhint %}

`GET https://developers.softone.gr/api/v1/sales?select=sales.documentCode,customerVatCode,customerId,createdAt`
