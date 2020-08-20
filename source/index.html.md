---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - python

search: true
---

# Introduction

Welcome to the Veridata API! Veridata is a modern, RESTful API-driven criminal record screening service. Veridata supports
searching NSOPW, the National Sex Offender Public Website, as well as various jurisdictions for county criminal records.

The Veridata API uses resource-oriented URLs, supports authentication and HTTPS verbs, and leverages JSON 
in all responses passed back to customers.

# Demo

Feel free to explore this interactive demo which provides search capabilities for criminal county court records in the state of Maryland. This demo purposely limits results to 20 records.
                                           
# Authentication

We support [HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) for all HTTP requests.

Veridata will provide you with an API key and an API token.

# NSOPW Search API

## NSOPW Search Request
Create a NSOPW (National Sex Offender Public Website) search request by providing the first name, last name, and date of birth.

```python
import requests 
import json

# Search Joe Shmoe with DOB 01/01/1970
search_type = "nsopw"
first_name = "Joe"
last_name = "Shmoe"
dob = "01/01/1970"
url = "https://api.veridata.io/v1/search?searchType={}&firstName={}&lastName={}&dob={}".format(search_type, first_name, last_name, dob)

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
  "request": {
    "request_id": "20200604X4IWOOFM"
  }
}
```

### HTTP Request

`GET https://api.veridata.io/v1/search`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
searchType | Yes | Should be set to "nsopw"
firstName | Yes | First name of the person to search
lastName | Yes | Last name of the person to search
dob | Yes |  Date of birth for the person to search

### Response

Column | Description
--------- | -------
request_id | Auto-generated, unique request identifier

See the Poll API below for details on fetching the response for the given request_id.

## Poll NSOPW Request
Using the request_id, poll Veridata to get the results of the request. If Veridata has not completed the request, the status of the request is returned.

```python
import requests 
import json

# Get the results for the previous county criminal request with request_id: 20200604X4IWOOFM
request_id = "20200604X4IWOOFM"

poll_url = "https://api.veridata.io/v1/records?request_id={}".format(request_id)

response = requests.get(poll_url, auth=('<INSERT API_KEY>', '<INSERT API_TOKEN>'))

# Get the JSON
contents = response.text
result = json.loads(contents)
```

> The above command returns status "processing" if the request is not complete:

```json
{
  "request": {
    "request_id": "20200604X4IWOOFM"
  },
  "status": "processing"
}
```

> If the request is complete, it returns the full response. In this case, one sex offender record match was found. 
> Note the age section will either type "yearOfBirth", "dateOfBirth", or "age" depending on the record,
> since some jurisdictions report the defendant's age differently.

```json
{
	"request": {
		"request_id": "20200604X4IWOOFM",
		"firstName": "Joe",
		"lastName": "Shmoe",
		"dob": "01/01/1970"
	},
	"records": [{
		"name": "Shmoe, Joe",
		"age": {
			"type": "dateOfBirth",
			"value": "01/01/1970"
		},
		"link": "https://www.mshp.dps.missouri.gov/CJ38/OffenderDetails?id=jk234n&x=32jk4-cd01-4ed4-234kj-6f3ef3eff324b"
	}],
	"offlineJurisdictions": null
}
```

> If the request was unable to be processed in the Veridata system, the following is returned:

```json
{
    "request": {
        "request_id": "20200604X4IWOOFM",
        "firstName": "Joe",
        "lastName": "Shmoe",
        "dob": "01/01/1970"
    },
	"failed_timestamp": "2020-06-04 05:05:54",
	"status": "failed",
	"note": "Please retry the request or contact your Veridata representative"
}
```

### HTTP Request

`GET https://api.veridata.io/v1/records`

### Query Parameters

Parameter | Description
--------- | -----------
request_id | Auto-generated, unique request identifier

### Response
The response contains the results of the NSOPW search, if Veridata is done processing the request.

The first section of the JSON with key `request` has the original request details.

The `records` section is a list of all the sex offender records that we identified based on the request details.

# Criminal County Search API

## Criminal County Search Request
Create a criminal county search request by providing the person's name, date of birth (optional), and county to search in.

We are also adding new capabilities to do custom filtering of the records. We currently support setting a filing date or a disposition date range to filter the records down.

```python
import requests 
import json

# Search Joe Shmoe in Allegany, Maryland
search_type = "countycriminal"
first_name = "Joe"
middle_name = ""
last_name = "Shmoe"
county = "Allegany"
state = "md"
filing_date = 7 # filter to last 7 years of records based on record filing date

url = "https://api.veridata.io/v1/search?searchType={}&" + 
"firstName={}&middleName={}&lastName={}&county={}&state={}%filingDate={}".format(search_type,
                                                                   first_name, 
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
  "request": {
    "request_id": "20200611QPB1BE11"
  }
}
```

### HTTP Request

`GET https://api.veridata.io/v1/search`

### Query Parameters

Parameter | Required | Description
--------- | ------- | -----------
searchType | Yes | Should be set to "countycriminal"
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

See the Poll API below for details on fetching the response for the given request_id.

## Poll Criminal County Request
Using the request_id, poll Veridata to get the results of the request. If Veridata has not completed the request, the status of the request is returned.

```python
import requests 
import json

# Get the results for the previous county criminal request with request_id: 20200611QPB1BE11
request_id = "20200611QPB1BE11"

poll_url = "https://api.veridata.io/v1/records?request_id={}".format(request_id)

response = requests.get(poll_url, auth=('<INSERT API_KEY>', '<INSERT API_TOKEN>'))

# Get the JSON
contents = response.text
result = json.loads(contents)
```

> The above command returns status "processing" if the request is not complete:

```json
{
  "request": {
    "request_id": "20200611QPB1BE11"
  },
  "status": "processing"
}
```

> If the request is complete, it returns the full response. In this case, one sex offender record match was found. 

```json
{
	"request": {
		"request_id": "20200611QPB1BE11",
		"firstName": "Joe",
		"middleName": null,
		"lastName": "Shmoe",
		"address": null,
		"dob": null,
		"county": "Allegany",
		"state": "MD"
	},
    "filtered_records": [{
            "defendant_information": {
                    "name": "Shmoe, Joe Alexander",
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
			"name": "Shmoe, Joe Alexander",
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
			"name": "Shmoe, Joe Peter",
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

> If the request was unable to be processed in the Veridata system, the following is returned:

```json
{
    "requests": {
        "request_id": "20200611QPB1BE11",
        "firstName": "Joe",
        "middleName": null,
        "lastName": "Shmoe",
        "address": null,
        "dob": null,
        "county": "Allegany",
        "state": "MD"
    },
	"failed_timestamp": "2020-06-11 03:05:23",
	"status": "failed",
	"note": "Please retry the request or contact your Veridata representative"
}
```


### HTTP Request

`GET https://api.veridata.io/v1/records`

### Query Parameters

Parameter | Description
--------- | -----------
request_id | Auto-generated, unique request identifier

### Response
The response contains the results of the county criminal search, if Veridata is done processing the request.

The first section of the JSON with key `request` has the original request details.

The `records` section is a list of all the criminal court records that we identified based on the request details.

Each `record` consists of the defendant information, as well a list of cases for that defendant.

Each `case` contains the case number, status, and charge-related information.

# Callback 
We now support callback API endpoints. Instead of polling the request_id as demonstrated in the previous section, 
we can directly send you the results of the request to an endpoint of your choosing.

Please work with your Veridata representative to enable this capability.
