# API Guidelines

## 1 Resources
There are two kinds of resources:
* **Collections** are a list of resources of the _same type_.
* A single **Resource** has some state (properties) and zero or more sub-resources.

### 1.1 URLs
URL scheme to access the resources is as follows:
* **Collections:** http://awesomeapi/resources _(e.g http://awesomeapi/cards)_
* **Resource:** http://awesomeapi/resource _(e.g. http://awesomeapi/cards/4321567812345678)_
* **Sub-resource:** http://awesomeapi/resource/resoure-id/sub-resource _(e.g. http://awesomeapi/customer/9876/cards)_

### 1.2 Naming conventions
#### 1.2.1 Resource URLs
As a rule of thumb, resource names must be **plural nouns** in most cases. E.g:
* http://awesomeapi/cards _(returns list of cards)_
* http://awesomeapi/customers/4210909 _(returns one customer with id 4210909)_
* http://awesomeapi/customers/4210909/cards _(returns list of cards for customer with ID 4210909)_

Verbs should only be used when we can't use a noun for that operation e.g. `calculate`.

#### 1.2.2 Resource properties (state)
Resource properties should follow **camelCase** convention e.g:
```
{
    "fullName": "Mehmood Ahmed",
    "homeAddress": "Gulshan-e-Iqbal, Karachi",
    ...
}
``` 

## 2 Standard methods
Each resource can support one or more of CRUD operations. These operations are often performed by means of HTTP verbs.

|Method|Http verb|Http request body|Http response body|
|-|-|-|-|
|List|GET \<collection URL>|Empty|Resource list 
|Get|GET \<resource URL>|Empty|Resource
|Create|POST \<collection URL>|Resource|Resource
|Update|PUT \<resource URL>|Resource|Resource
|Delete|DELETE \<resource URL>|Resource|Empty

### 2.1 Filtering
Filtering is mostly applicable on `List` method and **must** be done through **query string parameters**. E.g.
```
GET http://awesomeapi/customers/421909/accounts?type=current
```
The above call will return all Current accounts of customer with ID `421909`.

### 2.2 Pagination
#### 2.2.1 Request
Use `page` and `pageSize` parameters (**optional**) in query string for restricting number of records returned from API. E.g. 
```
GET http://awesomeapi/transactions?page=1&pageSize=20
``` 
will return _first 20 transactions_ from transactions resource.

#### 2.2.2 Response
`List` method response **must** have `values` and `totalSize` properties. The structure **must** comply to following:
```
{
    "values": [
        {...},
        {...}
    ],
    "totalSize": 1000, 
}
```

### 2.3 Examples
#### 2.3.1 List
```
GET http://awesomeapi/customers?page=2&pageSize=50
```
The response for the above call should look something like:
```
{
    "values": [
        { "cardNumber": "4565789009875432", ...},
        { "cardNumber": "4565789009875432", ...}
    ],
    "totalSize": 1000
}
```

#### 2.3.2 Get
```
GET http://awesomeapi/customers/421909
```
The response for the above call should look something like:
```
{
    "fullName": "Ahmed Khan",
    "phoneNumber": "...",
    ...,
}
```

### 2.4 Status codes
All successful operations **must** return HTTP `2xx` status code. For **Create** method, Http code `201` **must** be returned if resource is created successfully.

For more, refer to this [link](http://www.restapitutorial.com/httpstatuscodes.html).

## 3 Errors
Http error codes `4xx` must be used for error responses.

### 3.1 Error payload
Error response should be a json object containing an `error` property. This property is an object that **must** contain `code`, `message` properties in it. Optionally, this object contains a `target` property which is generally the name of the field that contains the error. There can be also be a `details` property that is a collection of `error` objects. E.g:
```
{
  "error": {
    "code": "BadArgument",
    "message": "Multiple errors in ContactInfo data",
    "target": "ContactInfo",
    "details": [
      {
        "code": "NullValue",
        "target": "PhoneNumber",
        "message": "Phone number must not be null"
      },
      {
        "code": "NullValue",
        "target": "LastName",
        "message": "Last name must not be null"
      },
      {
        "code": "MalformedValue",
        "target": "Address",
        "message": "Address is not valid"
      }
    ]
  }
}
``` 

