---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - python

search: true
---

# Introduction

Welcome to the Verified Data Services (VDS) API! VDS is a modern, RESTful API-driven county criminal screening service. 
The VDS API uses resource-oriented URLs, supports authentication and HTTPS verbs, and leverages JSON 
in all responses passed back to customers.
                                           
# Authentication

We support [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) for all HTTP requests.

VDS will provide you with an API key and an API token.

# Criminal Record Search API
The Criminal Record Search API provides a set of APIs to create county search requests as well as poll for the results of the requests.
We can also support a callback API if requested.

## Criminal County Search Request
Create a criminal search request.

```python
import requests 
import json

# Search James Smith in Allegany, Maryland
first_name = "James"
middle_name = ""
last_name = "Smith"
county = "Allegany"
state = "md"

url = "https://f7tw2gz7ij.execute-api.us-east-1.amazonaws.com/prod/v1/search?" + 
"firstName={}&middleName={}&lastName={}&county={}&state={}".format(first_name, 
                                                                   middle_name, 
                                                                   last_name, 
                                                                   county, 
                                                                   state)

# Make the request
response = requests.get(url, auth=('<INSERT API_KEY>', '<INSERT API_TOKEN>'))

# Get the JSON
contents = response.text
result = json.loads(contents)
```

<aside class="notice">
Make sure to replace the API key and API token.
</aside>

> The above command returns an auto-generated request ID. The JSON is structured like this:

```json
{
  "request_id": "CRZM78AUSI6W9BZ6E9K10QSYQYARW5SM"
}
```

### HTTP Request

`GET https://f7tw2gz7ij.execute-api.us-east-1.amazonaws.com/prod/v1/search`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
firstName | Yes | First name of the person to search
middleName | No | Middle name of the person to search
lastName | Yes | Last name of the person to search
county | Yes | County to search the individual in
state | Yes |  The corresponding state for the county to search the individual in

### Response

Column | Description
--------- | -------
request_id | Auto-generated, unique request identifier

## Poll County Search Request
Using the request_id, poll VDS to get the results of the request. If VDS has not completed the request, the status of the request is returned.

```python
import requests 
import json

# Get the results for the previous request with request_id: CRZM78AUSI6W9BZ6E9K10QSYQYARW5SM
request_id = "CRZM78AUSI6W9BZ6E9K10QSYQYARW5SM"

poll_url = "https://f7tw2gz7ij.execute-api.us-east-1.amazonaws.com/prod/v1/records?request_id={}".format(request_id)

response = requests.get(url, auth=('<INSERT API_KEY>', '<INSERT API_TOKEN>'))

# Get the JSON
contents = response.text
result = json.loads(contents)
```

> The above command either returns the status of the request:

```json
{
  "status": "processing"
}
```

> Or it returns the full response (in this case, two records were found):

```json
{
	"request": {
		"request_id": "CRZM78AUSI6W9BZ6E9K10QSYQYARW5SM",
		"firstName": "James",
		"middleName": null,
		"lastName": "Smith",
		"address": null,
		"dob": null,
		"county": "Allegany",
		"state": "MD"
	},
	"records": [{
		"defendant_information": {
			"name": "Smith, James Alexander",
			"dob": "01/01/1970",
			"address": "12345 Main Street, FLINTSTONE MD, 21530-0000",
			"race": "White",
			"sex": "Male",
			"eyes": null,
			"hair": null,
			"aliases": null
		},
		"cases": [{
			"case_number": "2W12334567",
			"case_status": "Closed",
			"county": "Allegany",
			"charges": [{
				"criminal_code": "1-1420",
				"disposition": "Nolle Prosequi",
				"disposition_date": "05/10/2002",
				"grade": "Felony Circuit Court",
				"charge_description": null
			}, {
				"criminal_code": "1-1415",
				"disposition": "Stet",
				"disposition_date": "09/02/2004",
				"grade": "Misdemeanor",
				"charge_description": null
			}, {
				"criminal_code": "1-1425",
				"disposition": "Stet",
				"disposition_date": "09/02/2004",
				"grade": "Misdemeanor",
				"charge_description": null
			}]
		}]
	}, {
		"defendant_information": {
			"name": "Smith, James Peter",
			"dob": "12/03/1962",
			"address": "123 Main Street, FLINTSTONE MD, 21532-0000",
			"race": "White",
			"sex": "Male",
			"eyes": null,
			"hair": null,
			"aliases": null
		},
		"cases": [{
			"case_number": "6W98765432",
			"case_status": "Closed",
			"county": "Allegany",
			"charges": [{
				"criminal_code": "3-2399",
				"disposition": "Stet",
				"disposition_date": "02/10/1995",
				"grade": "Misdemeanor",
				"charge_description": null
			}]
		}]
	}]
}
```

This endpoint retrieves the response for a given request_id

### HTTP Request

`GET https://f7tw2gz7ij.execute-api.us-east-1.amazonaws.com/prod/v1/records`

### Query Parameters

Parameter | Description
--------- | -----------
request_id | Auto-generated, unique request identifier

### Response
The response contains the results of the county criminal search, if VDS is done processing the request.

The first section of the JSON with key `request` has the original request details.

The `records` section is a list of all the criminal court records that we identified based on the request details.

Each `record` consists of the defendant information, as well a list of cases for that defendant.

Each `case` contains the case number, status, and charge-related information.
