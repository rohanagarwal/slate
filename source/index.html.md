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

# Demo

Feel free to explore our interactive [demo](http://demo.verifieddataservices.com) which provides search capabilities 
for criminal county court records in the state of Maryland.
                                           
# Authentication

We support [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) for all HTTP requests.

VDS will provide you with an API key and an API token.

# Criminal Record Search API
The Criminal Record Search API provides a set of APIs to create county search requests as well as poll for the results of the requests. We also support callback requests where we send the results of your search request to your API endpoint.

## Criminal County Search Request
Create a criminal county search request by providing the person's name, date of birth (optional), and county to search in.

We are also adding new capabilities to do custom filtering of the records. We currently support setting a filing date or a disposition date range to filter the records down.

```python
import requests 
import json

# Search John Smith in Allegany, Maryland
first_name = "John"
middle_name = ""
last_name = "Smith"
county = "Allegany"
state = "md"
filing_date = 7 # filter to last 7 years of records based on record filing date

url = "https://api.verifieddataservices.com/v1/search?" + 
"firstName={}&middleName={}&lastName={}&county={}&state={}%filingDate={}".format(first_name, 
                                                                   middle_name, 
                                                                   last_name, 
                                                                   county, 
                                                                   state,
                                                                   filing_date)

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

`GET https://api.verifieddataservices.com/v1/search`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
firstName | Yes | First name of the person to search
middleName | No | Middle name of the person to search
lastName | Yes | Last name of the person to search
county | Yes | County to search the individual in
state | Yes |  The corresponding state for the county to search the individual in
dateOfBirth | No |  Date of birth for the person to search
filingDate | No | Specify a number (in years) of records to filter down to. For example, 7 means that records with a filing date in the last 7 years will show up in the filtered section of the JSON response. 
dispositionDate | No | Specify a number (in years) of records to filter down to. For example, 7 means that records with a disposition date in the last 7 years will show up in the filtered section of the JSON response. 

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

poll_url = "https://api.verifieddataservices.com/v1/records?request_id={}".format(request_id)

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

> Or it returns the full response. In this case, two records were found, but there is one record in the filtered_records section because the filing date is within the last 7 years. Both records are in the records section of the JSON. 

```json
{
	"request": {
		"request_id": "CRZM78AUSI6W9BZ6E9K10QSYQYARW5SM",
		"firstName": "John",
		"middleName": null,
		"lastName": "Smith",
		"address": null,
		"dob": null,
		"county": "Allegany",
		"state": "MD"
	},
    "filtered_records": [{
            "defendant_information": {
                    "name": "Smith, John Alexander",
                    "dob": "01/01/1970", 
                    "address": "123 Main Street, FLINTSTONE MD, 21530-0000",
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
                    "filing_date": "01/01/2015",
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
    },
	"records": [{
		"defendant_information": {
			"name": "Smith, John Alexander",
			"dob": "01/01/1970",
			"address": "123 Main Street, FLINTSTONE MD, 21530-0000",
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
                        "filing_date": "01/01/2015",
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
			"name": "Smith, John Peter",
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
                        "filing_date": "01/01/2010",
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

`GET https://api.verifieddataservices.com/v1/records`

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

## Callback 
We now support callback API endpoints. Instead of polling the request_id as demonstrated in the previous section, 
we can directly send you the results of the request to an endpoint of your choosing.

Please work with your Veridata representative to enable this capability.
