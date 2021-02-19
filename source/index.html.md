---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - javascript
  - php
  # - ruby
  # - python

# toc_footers:
# - <a href='#'>Sign Up for a Developer Key</a>
# - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true
---

# Introduction

Welcome to the Canopy API! We follow standard behaviour in terms of URL's, JSON request/response bodies where applicable and standard HTTP error codes.

# Requesting Your Credentials

Credentials are provided on request by Canopy to yourselves. Please speak to your account manager here to obtain the details for the environments.

# Using API key

In the credentials you were sent you should have API key. Canopy expects for the API key to be included in all requests to the API in a header that looks like the following:

`x-api-key: ePYgsiGbWF5aAHBj0xT9Pa1k5li0NPMD25PQXbAC`

# Requesting an API Token

> To get API token, use this code:

```javascript
const jwt = require("jsonwebtoken");
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const clientId = "your_client_id";
const clientSecretKey = "your_secret_key";
const canopyApiKey = "your_api_key";

// Function that generates jwt token payload using clientId
const generatePayload = () => {
  const now = Math.floor(Date.now() / 1000); // sec
  const expires = now + 60 * 60;
  const payload = {
    iss: "canopy.rent",
    scope: "request.write_only document.read_only",
    aud: `referencing-requests/client/${clientId}/token`,
    exp: expires,
    iat: now,
  };

  return payload;
};

const authenticate = async () => {
  // Generate jwt key
  const jwtKey = await jwt.sign(generatePayload(), clientSecretKey);

  // Send POST request
  return axios({
    url: `${canopyEndpointBaseUri}/referencing-requests/client/${clientId}/a`,
    method: "POST",
    headers: {
      // Add API key from the credentials into headers
      "x-api-key": canopyApiKey,
    },
    data: {
      jwtKey: jwtKey,
    },
  });
};

authenticate();
```

```php
<?php
require_once 'vendor/autoload.php';

// For JWT generation we used Firebase PHP-JWT library: https://github.com/firebase/php-jwt
use \Firebase\JWT\JWT;

$clientId = "your_client_id";
$secretKey = "your_secret_key";

$now = time();

// Generate JWT payload
$payload = array(
    "iss" => "canopy.rent",
    "scope" => "request.write_only document.read_only",
    "aud" => "referencing-requests/client/$clientId/token",
    "exp" => $now + 60 * 60,
    "iat" => $now
);

// Generate JWT key using payload and secret key
$jwt = JWT::encode($payload, $secretKey);

echo $jwt, "\n";

$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";

// We used Guzzle as HTTP client: https://docs.guzzlephp.org/en/stable/#
$client = new \GuzzleHttp\Client();

// Send POST request to get access token with your api key and generated JWT key
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/client/$clientId/token",
  [
    "headers" => [
      "Content-Type" => "application/json",
      "x-api-key" => $apiKey
    ],
    "json" => ["jwtKey" => $jwt]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "access_token": "Bearer eyaasd456FFGDFGdfgdfgdfgdfg7sdyfg35htjl3bhef89y4rjkbergv-KAj-dGh0xEuZftO_Utm6dugKQ",
  "expires": 1594907203
}
```

### HTTP Request

`POST /referencing-requests/client/:clientId/token`

### Body Parameters

| Parameter | Type   | Required | Description       |
| --------- | ------ | -------- | ----------------- |
| jwtKey    | string | true     | Generated jwt key |

### Using API token

You need to request an API token to make calls to the Canopy API. Canopy expects for the API token to be included in all requests to the API in a header that looks like the following:

`Authorization: Bearer eyaasd456FFGDFGdfgdfgdfgdfg7sdyfg35htjl3bhef89y4rjkbergv-KAj-dGh0xEuZftO_Utm6dugKQ`

# Refresh Secret Key

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client/refresh`,
  method: "POST",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
  data: {
    secretKey: "new_secret_key",
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/client/refresh",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => ["secretKey" => "new_secret_key"]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "secretKey": "new_secret_key"
  }
}
```

This endpoint may be called to change current `secretKey` (which is required for `jwtKey` generation) and get back freshly generated one. Accepts currently active `secretKey` in request body. After receiving successful response with new `secretKey`, previous `secretKey` becomes stale and should not be used for `jwtKey` generation.

### HTTP Request

`POST /referencing-requests/client/refresh`

### Body Parameters

| Parameter | Type   | Required | Description                 |
| --------- | ------ | -------- | --------------------------- |
| secretKey | string | true     | Currently active secret key |

# Prefill User's Rent Passport with Existing Data

If you already have some user information, such as identity, income and (or) rent, you can send it Canopy, so we will prefill the user's Rent Passport with provided data.

## Set Identity

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client-user-data/set-identity`,
  method: "POST",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
  data: {
    email: "example@email.com",
    firstName: "FirstName",
    middleName: "MiddleName",
    lastName: "LastName",
    dateOfBirth: "2000-03-05",
    phone: "88002553535",
    addresses: [
      {
        startDate: "2020-09-01",
        flat: "9",
        houseNumber: "113",
        street: "Street Name",
        countryCode: "GB",
        town: "London",
        postCode: "M1 1AE"
      }
    ]
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/client-user-data/set-identity",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => [
      "email" => "example@email.com",
      "firstName" => "FirstName",
      "middleName" => "MiddleName",
      "lastName" => "LastName",
      "dateOfBirth" => "2000-03-05",
      "phone" => "88002553535",
      "addresses" => [
        [
          "startDate" => "2020-09-01",
          "flat" => "9",
          "houseNumber" => "113",
          "street" => "Street Name",
          "countryCode" => "GB",
          "town" => "London",
          "postCode" => "M1 1AE"
        ]
      ]
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "email": "example@email.com",
    "firstName": "FirstName",
    "middleName": "MiddleName",
    "lastName": "LastName",
    "dateOfBirth": "2000-03-05",
    "phone": "88002553535",
    "addresses": [
      {
        "startDate": "2020-09-01",
        "flat": "9",
        "houseNumber": "113",
        "street": "Newton Street",
        "countryCode": "GB",
        "town": "Manchester",
        "postCode": "M1 1AE"
      }
    ]
  }
}
```

### HTTP Request

`POST /referencing-requests/client-user-data/set-identity`

### Body Parameters

| Parameter   | Type   | Required | Description                                          |
| ----------- | ------ | -------- | ---------------------------------------------------- |
| email       | string | true     |                                                      |
| firstName   | string | true     |                                                      |
| middleName  | string | false    |                                                      |
| lastName    | string | true     |                                                      |
| dateOfBirth | string | true     | Date format: YYYY-MM-DD                              |
| phone       | string | true     |                                                      |
| addresses   | array  | true     | Array of addresses. See address data structure below |

### Address data structure

| Parameter   | Type   | Required                           | Description                                                                                                                             |
| ----------- | ------ | ---------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| startDate   | string | true                               | Date format: YYYY-MM-DD                                                                                                                 |
| flat        | string | false                              |                                                                                                                                         |
| houseNumber | string | true if houseName is not present   |                                                                                                                                         |
| houseName   | string | true if houseNumber is not present |                                                                                                                                         |
| street      | string | true                               |                                                                                                                                         |
| countryCode | string | true                               | You can see list of available country codes [in our apidocs repository](https://github.com/insurestreetltd/canopy-apidocs#set-identity) |
| county      | string | false                              |                                                                                                                                         |
| town        | string | true                               |                                                                                                                                         |
| postCode    | string | true                               |                                                                                                                                         |
| line1       | string | false                              |                                                                                                                                         |
| line2       | string | false                              |                                                                                                                                         |
| line3       | string | false                              |                                                                                                                                         |

## Set Income

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client-user-data/set-income`,
  method: "POST",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
  data: {
    email: "example@email.com",
    data: {
      income: [
        {
          incomeSource: "EMPLOYED",
          annualSalary: 100000,
          paymentFrequency: "MONTHLY",
          employment: {
            companyName: "Company name",
            jobTitle: "Job title",
            employmentStatus: "FULL_TIME",
            employmentBasis: "CONTRACT",
            startDate: "2020-09-01"
          }
        }
      ]
    }
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/client-user-data/set-income",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => [
      "email" => "example@email.com",
      "data" => [
        "income" => [
          [
            "incomeSource" => "EMPLOYED",
            "annualSalary" => 100000,
            "paymentFrequency" => "MONTHLY",
            "employment" => [
              "companyName" => "Company name",
              "jobTitle" => "Job title",
              "employmentStatus" => "FULL_TIME",
              "employmentBasis" => "CONTRACT",
              "startDate" => "2020-09-01"
            ]
          ]
        ]
      ]
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "clientId": "clientId",
    "email": "example@email.com",
    "income": [
      {
        "incomeSource": "EMPLOYED",
        "annualSalary": 100000,
        "paymentFrequency": "MONTHLY",
        "employment": {
          "companyName": "Comapny name",
          "jobTitle": "Job title",
          "employmentStatus": "FULL_TIME",
          "employmentBasis": "CONTRACT",
          "startDate": "2020-09-01"
        }
      }
    ]
  }
}
```

### HTTP Request

`POST /referencing-requests/client-user-data/set-income`

### Body Parameters

| Parameter | Type   | Required | Description                |
| --------- | ------ | -------- | -------------------------- |
| email     | string | true     |                            |
| data      | object | true     | Object with `income` field |

`data` field structure:

| Parameter | Type  | Required | Description                                                                                     |
| --------- | ----- | -------- | ----------------------------------------------------------------------------------------------- |
| income    | array | true     | Array on income objects. Must contain minimum 1 item. See list of possible income objects below |

### Possible income objects

<b>"EMPLOYED" type:</b>

| Parameter        | Type       | Required | Description                                                       |
| ---------------- | ---------- | -------- | ----------------------------------------------------------------- |
| incomeSource     | "EMPLOYED" | true     | Income source field with value "EMPLOYED"                         |
| annualSalary     | integer    | true     |                                                                   |
| paymentFrequency | string     | true     | One of ["MONTHLY", "TWO_WEEKLY", "WEEKLY"]                        |
| additionalInfo   | string     | false    |                                                                   |
| employment       | object     | true     | Object with employment data. See employment field structure below |

Employment object structure for income source with "EMPLOYED" type:

| Parameter        | Type   | Required | Description                       |
| ---------------- | ------ | -------- | --------------------------------- |
| companyName      | string | true     |                                   |
| jobTitle         | string | true     |                                   |
| employmentStatus | string | true     | One of ["FULL_TIME", "PART_TIME"] |
| employmentBasis  | string | true     | One of ["PERMANENT", "CONTRACT"]  |
| startDate        | string | true     | Date format: YYYY-MM-DD           |

<b>"SELF_EMPLOYED" type:</b>

| Parameter        | Type            | Required | Description                                                       |
| ---------------- | --------------- | -------- | ----------------------------------------------------------------- |
| incomeSource     | "SELF_EMPLOYED" | true     | Income source field with value "SELF_EMPLOYED"                    |
| annualSalary     | integer         | true     |                                                                   |
| paymentFrequency | string          | true     | One of ["MONTHLY", "TWO_WEEKLY", "WEEKLY"]                        |
| additionalInfo   | string          | false    |                                                                   |
| employment       | object          | true     | Object with employment data. See employment field structure below |

Employment object structure for income source with "SELF_EMPLOYED" type:

| Parameter  | Type   | Required | Description             |
| ---------- | ------ | -------- | ----------------------- |
| incomeName | string | true     |                         |
| jobTitle   | string | true     |                         |
| startDate  | string | true     | Date format: YYYY-MM-DD |

<b>"STUDENT" type:</b>

| Parameter        | Type      | Required | Description                                                       |
| ---------------- | --------- | -------- | ----------------------------------------------------------------- |
| incomeSource     | "STUDENT" | true     | Income source field with value "STUDENT"                          |
| annualSalary     | integer   | true     |                                                                   |
| paymentFrequency | string    | true     | One of ["MONTHLY", "TWO_WEEKLY", "WEEKLY", "OTHER"]               |
| additionalInfo   | string    | false    |                                                                   |
| employment       | object    | true     | Object with employment data. See employment field structure below |

Employment object structure for income source with "EMPLOYED" type:

| Parameter                  | Type    | Required | Description             |
| -------------------------- | ------- | -------- | ----------------------- |
| educationalInstitutionName | string  | true     |                         |
| grantsAvailability         | boolean | true     |                         |
| startDate                  | string  | true     | Date format: YYYY-MM-DD |

<b>"RETIRED", "UNEMPLOYED", "BENEFITS" or "OTHER" type</b>

| Parameter        | Type    | Required | Description                                                       |
| ---------------- | ------- | -------- | ----------------------------------------------------------------- |
| incomeSource     | string  | true     | One of ["RETIRED", "UNEMPLOYED", "BENEFITS", "OTHER"]             |
| annualSalary     | integer | true     |                                                                   |
| paymentFrequency | string  | true     | One of ["MONTHLY", "TWO_WEEKLY", "WEEKLY", "OTHER"]               |
| additionalInfo   | string  | false    |                                                                   |
| employment       | object  | true     | Object with employment data. See employment field structure below |

Employment object structure for income source with "EMPLOYED" type:

| Parameter   | Type   | Required | Description             |
| ----------- | ------ | -------- | ----------------------- |
| incomeName  | string | true     |                         |
| description | string | true     |                         |
| startDate   | string | true     | Date format: YYYY-MM-DD |

## Set Rent

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client-user-data/set-rent`,
  method: "POST",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
  data: {
    email: "example@email.com",
    data: {
      homeowner: false,
      rentsDuringLastYear: true,
      rents: [
        {
          name: "Rent name",
          paymentFrequency: "MONTHLY",
          rentPaymentAmount: 10000,
          rentPaidTo: "Paid to"
        }
      ]
    }
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/client-user-data/set-rent",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => [
      "email" => "example@email.com",
      "data" => [
        "homeowner" => false,
        "rentsDuringLastYear" => true,
        "rents" => [
          [
            "name" => "Rent name",
            "paymentFrequency" => "MONTHLY",
            "rentPaymentAmount" => 10000,
            "rentPaidTo" => "Paid to"
          ]
        ]
      ]
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "clientId": "clientId",
    "email": "example@email.com",
    "rentData": {
      "homeowner": false,
      "rentsDuringLastYear": true,
      "rents": [
        {
          "name": "Rent name",
          "paymentFrequency": "MONTHLY",
          "rentPaymentAmount": 10000,
          "rentPaidTo": "Paid to"
        }
      ]
    }
  }
}
```

### HTTP Request

`POST /referencing-requests/client-user-data/set-rent`

### Body Parameters

| Parameter | Type   | Required | Description           |
| --------- | ------ | -------- | --------------------- |
| email     | string | true     |                       |
| data      | object | true     | Object with rent data |

### data field structure

| Parameter           | Type    | Required                            | Description                                     |
| ------------------- | ------- | ----------------------------------- | ----------------------------------------------- |
| homeowner           | boolean | true                                |                                                 |
| rentsDuringLastYear | boolean | true                                | Object with rent data                           |
| rents               | array   | true if rentsDuringLastYear is true | Array of rents during last year. Minimum 1 item |

### Rents item structure

| Parameter         | Type    | Required | Description                                         |
| ----------------- | ------- | -------- | --------------------------------------------------- |
| name              | string  | true     |                                                     |
| paymentFrequency  | string  | true     | One of ["MONTHLY", "TWO_WEEKLY", "WEEKLY", "OTHER"] |
| rentPaymentAmount | integer | true     |                                                     |
| rentPaidTo        | string  | true     |                                                     |

# Referencing Endpoints

Inside the Canopy system, there are 3 main entities: company, branch and agent. All communications between the agents and renters happen on the branch level, not on the company level. It means that each company must have at least one branch (let’s call it _default_), so all existing or new agents should belong to the particular branch(es) in order to work with the potential or actual renters. For example, if the agent belongs to **Branch 1** and this agent sends the invite to the potential renter, actually the invite is sent from **Branch 1**, not directly from the agent or **Company**.

<p align="center"><img src="/images/company-branch.svg" /></p>

Therefore, in order to work with us, we create a Company and the default Branch for you at the very beginning. On the basic level, it’s enough for the correct workflow. There are the following possible scenarios:

- **You work on the Company level and don’t have such concept as Branch**: </br>
  In that case, there is no need to Link your branches with Canopy branches. For all the internal interactions, we will use the default branch (**Branch 1**) created for you.
- **Beside the Company, you have one Branch as well so you work on the Branch level as we do**: </br>
  In that case, you can link your single branch with our default branch, but it’s also an optional step. Anyway, we will use the default **Branch 1** for our internal communication with the renters.
- **Beside the Company, you have several branches so you work on the Branch level as we do:** </br>
  This is basically the same approach which is used inside the Canopy system. In order to request a reference for the particular renter, you can either:

  - Use the default branch, which is already created in the Canopy
  - Preliminary create an additional branches (Branch 2, Branch 3, …, Branch N) in Canopy system and then link/map them with your own branches.

    </br>After that, at the moment of requesting a reference from Canopy for the user, you can specify the branch to which this reference should be attached. But again, this step of branch mapping is an optional step. If you don’t want to link anything, just skip the Branch Linking step so that we will always use the default branch for the internal interactions with renter.

## Get the List of Branches and Connections

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const clientId = "client_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client/${clientId}/branches-list`,
  method: "GET",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$clientId = "your_client_id";
$authorizationToken = "authorization_token";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "GET",
  "$canopyEndpointBaseUri/referencing-requests/client/$clientId/branches-list",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "branches": [
      {
        "canopyBranchId": "592954da-4ea7-426b-ace3-f61dbc169ea4",
        "clientBranchId": "592954da-4ea7-426b-ace3-f61dbc169ea4",
        "branchName": "Branch 1",
        "branchAddress": {
          "id": "af380cca-e2a6-4cc2-8abc-99889f4e787f",
          "line1": "Apartment 9",
          "line2": "113 Street name",
          "line3": null,
          "street": "Street name",
          "town": "London",
          "postCode": "M1 1AE",
          "countryCode": "GB",
          "flat": "9",
          "district": null,
          "houseNumber": "113",
          "houseName": null
        }
      },
      {
        "canopyBranchId": "592954da-4ea7-426b-ace3-f61dbc169ea4",
        "clientBranchId": "0830f800-b5d2-4e16-aede-411cb5f3c58b",
        "branchName": "Branch 2",
        "branchAddress": {
          "id": "af380cca-e2a6-4cc2-8abc-99889f4e787f",
          "line1": "Apartment 9",
          "line2": "113 Street name",
          "line3": null,
          "street": "Street name",
          "town": "London",
          "postCode": "M1 1AE",
          "countryCode": "GB",
          "flat": "9",
          "district": null,
          "houseNumber": "113",
          "houseName": null
        }
      },
      {
        "canopyBranchId": "818e8ad7-97aa-460d-a007-6b65830ace5b",
        "clientBranchId": null,
        "branchName": "Branch 3",
        "branchAddress": {
          "id": "af380cca-e2a6-4cc2-8abc-99889f4e787f",
          "line1": "Apartment 9",
          "line2": "113 Street name",
          "line3": null,
          "street": "Street name",
          "town": "London",
          "postCode": "M1 1AE",
          "countryCode": "GB",
          "flat": "9",
          "district": null,
          "houseNumber": "113",
          "houseName": null
        }
      }
    ]
  }
}
```

The endpoint below returns the list of canopy branches associated with `clientId` and connections with client branches. Canopy branch might be linked with some client branch, in this case `clientBranchId` value will contain id of the client branch. If canopy branch linked with multiple client branches, then same canopy branch will appear in the list multiple times with different `clientBranchId`. If canopy branch is not linked to any client branch, then `clientBranchId` will be equal to `null`.

### HTTP Request

`GET /referencing-requests/client/:clientId/branches-list`

### URL Parameters

| Parameter | Description           |
| --------- | --------------------- |
| clientId  | Your client reference |

### Response Structure

| Parameter | Type   | Description                                               |
| --------- | ------ | --------------------------------------------------------- |
| success   | bool   | Request status                                            |
| requestId | string | Request id                                                |
| data      | object | Object that contains branches field with list of branches |

Data object structure:

| Parameter | Type  | Description      |
| --------- | ----- | ---------------- |
| branches  | array | List of branches |

Single branch data structure:

| Parameter      | Type           | Description                                                                                                                                          |
| -------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| canopyBranchId | string         | id of the branch in the Canopy system                                                                                                                |
| clientBranchId | string or null | id of the client's branch that is linked with this canopy branch, if equals null than this canopy branch is not linked with any client branch branch |
| branchName     | string         | The name of the Canopy branch                                                                                                                        |
| branchAddress  | object         | Object that contains information about branch address. You can see example of the address structure in the code example                              |

## Link Branch

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const clientId = "client_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client/${clientId}/link-branch`,
  method: "POST",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
  data: {
    canopyBranchId: "592954da-4ea7-426b-ace3-f61dbc169ea4",
    clientBranchId: "592954da-4ea7-426b-ace3-f61dbc169ea4"
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$clientId = "your_client_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/client/$clientId/link-branch",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => [
      "canopyBranchId" => "592954da-4ea7-426b-ace3-f61dbc169ea4",
      "clientBranchId" => "592954da-4ea7-426b-ace3-f61dbc169ea4"
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "canopyBranchId": "592954da-4ea7-426b-ace3-f61dbc169ea4",
    "clientBranchId": "592954da-4ea7-426b-ace3-f61dbc169ea4",
    "branchName": "Branch 1",
    "branchAddress": {
      "id": "af380cca-e2a6-4cc2-8abc-99889f4e787f",
      "line1": "Apartment 9",
      "line2": "113 Street name",
      "line3": null,
      "street": "Street name",
      "town": "London",
      "postCode": "M1 1AE",
      "countryCode": "GB",
      "flat": "9",
      "district": null,
      "houseNumber": "113",
      "houseName": null
    }
  }
}
```

Thish endpoint links existing Canopy branch with client's branch. This operation is required for [referencing request](https://insurestreetltd.github.io/canopy-apidocs/#request-referencing). `clientBranchId` should be unique, so there can be only one connection with specific client branch. `canopyBranchId` can be used in multiple connections, so there can be multiple connections between single canopy branch and multiple client's branches.

### HTTP Request

`POST /referencing-requests/client/:clientId/link-branch`

### URL Parameters

| Parameter | Description           |
| --------- | --------------------- |
| clientId  | Your client reference |

### Body Parameters

| Parameter      | Type   | Description                                                                                                                                                                                    |
| -------------- | ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| canopyBranchId | string | id of the Canopy branch, you can get list of possible branches using [GET /bracnhes-list endpoint](https://insurestreetltd.github.io/canopy-apidocs/#get-the-list-of-branches-and-connections) |
| clientBranchId | string | id of the branch in the client's system, it can be any string of uuid format and should be unique                                                                                              |

## Delete Branch Mapping

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const clientId = "client_id";
const clientBranchId = "client_branch_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client/${clientId}/branch-mapping/${clientBranchId}`,
  method: "DELETE",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  }
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$clientId = "your_client_id";
$clientBranchId = "client_branch_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "DELETE",
  "$canopyEndpointBaseUri/referencing-requests/client/$clientId/branch-mapping/$clientBranchId",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "data": {
    "clientBranchId": "592954da-4ea7-426b-ace3-f61dbc169ea4"
  }
}
```

This endpoint deletes existing branch mapping between Canopy branch and client branch.

### HTTP Request

`DELETE /referencing-requests/client/:clientId/branch-mapping/:clientBranchId`

### URL Parameters

| Parameter      | Description                                    |
| -------------- | ---------------------------------------------- |
| clientId       | Your client reference                          |
| clientBranchId | id of the branch linked with the Canopy branch |

## Request Referencing

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const clientId = "client_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client/${clientId}/request`,
  method: "POST",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
  data: {
    email: "test@email.com",
    firstName: "First name",
    lastName: "Last name",
    callbackUrl: "https://callbackurl.com",
    requestType: "RENTER_SCREENING",
    itemType: "FULL",
    title: "Title",
    phone: "88002553535",
    branchId: "branch_id",
    clientReferenceId: "12346"
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$clientId = "your_client_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/client/$clientId/request",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => [
      "email" => "test@email.com",
      "firstName" => "First name",
      "lastName" => "Last name",
      "callbackUrl" => "https://callbackurl.com",
      "requestType" => "RENTER_SCREENING",
      "itemType" => "FULL",
      "title" => "Title",
      "phone" => "88002553535",
      "branchId" => "branch_id",
      "clientReferenceId" => "12346"
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "requestId": "request_id",
  "canopyReferenceId": "canopy_reference_id"
}
```

This endpoint creates new referencing request.

### HTTP Request

`POST /referencing-requests/client/:clientId/request`

### URL Parameters

| Parameter | Description           |
| --------- | --------------------- |
| clientId  | Your client reference |

### Body Parameters

| Parameter         | Type    | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ----------------- | ------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| email             | string  | true     |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| firstName         | string  | false    |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| lastName          | string  | false    |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| callbackUrl       | string  | true     | URL to which Canopy will send PDF Report                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| requestType       | string  | true     | One of ["RENTER_SCREENING", "GUARANTOR_SCREENING"]                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| itemType          | string  | true     | One of ["INSTANT", "FULL"]                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| title             | string  | false    | Title that is used before a surname or full name                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| phone             | string  | false    |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| branchId          | string  | false    | clientBranchId that is connected to any canopy branch, you can get list of created connections using [GET /bracnhes-list endpoint](https://insurestreetltd.github.io/canopy-apidocs/#get-the-list-of-branches-and-connections), if there's no connection created with such branchId then this API call will return 404 error, new connection can be created using [POST /link-branch endpoint](https://insurestreetltd.github.io/canopy-apidocs/#link-branch), if no branchId is passed than default branch will be used for the referencing request |
| clientReferenceId | string  | false    | A unique identifier on the client's side                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| isGuarantorNeeded | boolean | false    | Flag allowing to request a guarantor for a specified user (user email in request). Applicable only if requestType = "RENTER_SCREENING"                                                                                                                                                                                                                                                                                                                                                                                                               |

## Rent Passport Retrieval

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const clientId = "client_id";
const documentId = "document_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client/${clientId}/documents/${documentId}`,
  method: "GET",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$clientId = "your_client_id";
$documentId = "document_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "GET",
  "$canopyEndpointBaseUri/referencing-requests/client/$clientId/documents/$documentId",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

Once referencing has been completed by the renter in the Canopy mobile application, an update message will be sent to the `callbackUrl` provided at the moment of referencing request. This message will have the following structure:

| Parameter         | Type   | Description                                                    |
| ----------------- | ------ | -------------------------------------------------------------- |
| clientReferenceId | string |                                                                |
| canopyReferenceId | string |                                                                |
| document          | object | Object with document data. See document object structure below |

Document data structure:

| Parameter            | Type    | Description                                                                 |
| -------------------- | ------- | --------------------------------------------------------------------------- |
| documentType         | integer | One of [0, 1]. 0 means INSTANT screening type, 1 means FULL screening type. |
| url                  | string  | `/referencing-requests/client/:clientId/documents/:documentId`              |
| maxRent              | integer |                                                                             |
| status               | string  |                                                                             |
| title                | string  | One of [INSTANT, FULL]                                                      |
| isGuarantorReference | boolean | Exists if requestType of reference request was GUARANTOR_SCREENING          |
| isGuarantorNeeded    | boolean | Exists if requestType of reference request was RENTER_SCREENING             |
| waitingForGuarantor  | boolean | Exists if isGuarantorNeeded has true value                                  |
| guarantorCompleteRP  | boolean | Exists if isGuarantorNeeded has true value                                  |

- If `requestType` of reference request is `GUARANTOR_SCREENING`, after the user completes his referencing, the Report will be generated as for the Guarantor.
- If `isGuarantorNeeded` of reference request is `true`, we will send 1 or 2 Reports:
  - First case (sending only one Report): Renter and guarantor have completed their Referencings - we send a Report with both data (renter and guarantor).
  - Second case (sending two Reports): Renter has completed Referencing, but the guarantor is still not present - we send the Report only with the user's data. After the Guarantor completes the referencing, we will send one more Report with both data (renter and guarantor).

To retrieve a PDF Rent Passport after successful referencing completion, you can use the url field from the above response:

### HTTP Request

`GET /referencing-requests/client/:clientId/documents/:documentId`

### URL Parameters

| Parameter  | Description                                              |
| ---------- | -------------------------------------------------------- |
| clientId   | Your client reference                                    |
| documentId | The ID of the document returned from the document update |

The PDF will be included in the response body with the filename specified in the header as follows:

`content-type: application/pdf` <br>
`content-disposition: attachment; filename='9e6222ee-312b-2297-a03a-07700363a6b6_FULL.pdf' `

## Get Intermediate PDF Report

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const clientId = "client_id";
const canopyReferenceId = "canopy_reference_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client/${clientId}/rent-passport/${canopyReferenceId}`,
  method: "GET",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$clientId = "your_client_id";
$canopyReferenceId = "canopy_reference_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "GET",
  "$canopyEndpointBaseUri/referencing-requests/client/$clientId/rent-passport/$canopyReferenceId",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

Get Screening Results of the renter, even if Passport is not completed. When you receive notifications that some of the Renter Passports sections were updated, you can call and get a PDF Report with current state. For example it is useful when renter should provide FULL referencing, but you wish to see INSTANT screening results as soon as it is completed and not wait while full screening will be passed. If `requestType` of last reference request for user is `GUARANTOR_SCREENING`, the Report will be generated as for the Guarantor.

### HTTP Request

`GET /referencing-requests/client/:clientId/rent-passport/:canopyReferenceId`

### URL Parameters

| Parameter         | Description                                     |
| ----------------- | ----------------------------------------------- |
| clientId          | Your client reference                           |
| canopyReferenceId | The id of registered request on the Canopy side |

Also, this endpoint may accept ?format=json query param to return RP in json format, in other cases returns pdf.

## Revoke Guarantor Request

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const clientId = "client_id";
const webhookType = "webhook_type";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client/${clientId}/revoke-guarantor-request`,
  method: "DELETE",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
  data: {
    email: "example@email.com",
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$clientId = "your_client_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "DELETE",
  "$canopyEndpointBaseUri/referencing-requests/client/$clientId/revoke-guarantor-request",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => [
      "email" => "test@email.com"
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true
}
```

This endpoint revokes guarantor request for the specified user.

### HTTP Request

`DELETE /referencing-requests/client/:clientId/revoke-guarantor-request`

### URL Parameters

| Parameter | Description           |
| --------- | --------------------- |
| clientId  | Your client reference |

### Body Parameters

| Parameter | Type   | Required | Description |
| --------- | ------ | -------- | ----------- |
| email     | string | true     |             |

# Webhooks Endpoints

## Register Webhook

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const clientId = "client_id";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client/${clientId}/webhook/register`,
  method: "POST",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  },
  data: {
    type: "PASSPORT_STATUS_UPDATES",
    callbackUrl: "https://callbackurl.com",
    additionalSettings: ["INCOME", "RENT"]
  },
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$clientId = "your_client_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "POST",
  "$canopyEndpointBaseUri/referencing-requests/client/$clientId/webhook/register",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ],
    "json" => [
      "type" => "PASSPORT_STATUS_UPDATES",
      "callbackUrl" => "https://callbackurl.com",
      "additionalSettings" => ["INCOME", "RENT"]
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "webhookType": "PASSPORT_STATUS_UPDATES",
  "callbackUrl": "https://callbackurl.com"
}
```

This endpoint registers the webhook with the appropriate type in Canopy system. After that, Rent Passport updates will be sent to the specified callback url.

### HTTP Request

`POST /referencing-requests/client/:clientId/webhook/register`

### URL Parameters

| Parameter | Description           |
| --------- | --------------------- |
| clientId  | Your client reference |

### Body Parameters

| Parameter          | Type     | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ------------------ | -------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type               | string   | true     | Specifies which type of updates will be sent from Canopy. Could be one of ["PASSPORT_STATUS_UPDATES", "REQUEST_STATUS_UPDATES"].                                                                                                                                                                                                                                                                                                                                           |
| callbackUrl        | string   | true     | The URL of the webhook endpoint                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| additionalSettings | string[] | false    | List of Rent Passport sections for which updates will be sent. This field should only be added in case "PASSPORT_STATUS_UPDATES" webhook type is specified. The following sections can be specified inside the array: ["INCOME", "RENT", "CREDIT_CHECK", "SAVINGS", "RENTAL_PREFERENCES", "EMPLOYEE_REFERENCE", "LANDLORD_REFERENCE"]. There is no need to explicitly specify the names of all sections if you want to receive all updates, just send an empty array - []. |

### Event types

If you subscribed to the `REQUEST_STATUS_UPDATES` type, the updates will be sent to the `callbackUrl` each time one of the following events trigger:

| Event                                      | Description                                                                                                                                                                                                       |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `INVITED`                                  | The user was invited to connect                                                                                                                                                                                   |
| `INVITE_RESENT`                            | Resent invitation email for the user                                                                                                                                                                              |
| `CONNECTED`                                | User accepts the connection                                                                                                                                                                                       |
| `CONNECTION_REJECTED`                      | User rejects the connection                                                                                                                                                                                       |
| `CONNECTION_STOPPED`                       | User stops the connection                                                                                                                                                                                         |
| `SENDING_COMPLETED_PASSPORT_FAILED`        | Sending the completed passport to client failed                                                                                                                                                                   |
| `PASSPORT_COMPLETED`                       | User complete his passport and the document was sent. If a guarantor was requested for this user, the status means that user and guarantor complete their passports and the document with both passports was sent |
| `INVALID_APPLICATION_DETAILS`              | Client's request body with application details was invalid                                                                                                                                                        |
| `PASSPORT_COMPLETED_WAITING_FOR_GUARANTOR` | User complete his passport and the document was sent, but the guarantor requested for this user did not complete his passport                                                                                     |
| `GUARANTOR_REQUEST_CANCELLED`              | The guarantor request for the specified user was cancelled                                                                                                                                                        |

Once `REQUEST_STATUS_UPDATES` event trigger, the Canopy should sent the notification to `callbackUrl` in the following format:

| Parameter         | Type   |
| ----------------- | ------ |
| canopyReferenceId | string |
| clientReferenceId | string |
| notes             | string |

If you are subscribed to the `PASSPORT_STATUS_UPDATES` type, the updates will be sent to the `callbackUrl` when one of the following Rent Passport sections is updated (if updates for this section requested):

| Section              | Description                                  |
| -------------------- | -------------------------------------------- |
| `CREDIT CHECK`       |                                              |
| `INCOME`             |                                              |
| `RENT`               |                                              |
| `SAVINGS`            |                                              |
| `LANDLORD REFERENCE` | optional, only if `FULL SCREENING` requested |
| `EMPLOYER REFERENCE` | optional, only if `FULL SCREENING` requested |

### Notification format

Once `PASSPORT_STATUS_UPDATES` event trigger, the Canopy should sent the notification to `callbackUrl` in the following format:

| Parameter         | Type   | Description                                                                              |
| ----------------- | ------ | ---------------------------------------------------------------------------------------- |
| canopyReferenceId | string |                                                                                          |
| clientReferenceId | string |                                                                                          |
| updatedSection    | object | Object with updated section data. See `updatedSection` data structure below              |
| instant           | object | Object with instant data. See `instant` data structure below                             |
| full              | object | Optional field, sent only if `FULL SCREENING` requested. See `full` data structure below |
| globalStatus      | string | One of ["NOT_STARTED", "IN_PROGRESS", "ACCEPT", "CONSIDER", "HIGH_RISK"]                 |

`updatedSection` data structure:

| Parameter | Type        | Description                                                                                      |
| --------- | ----------- | ------------------------------------------------------------------------------------------------ |
| type      | string      | One of ["INCOME", "RENT", "CREDIT_CHECK", "SAVINGS", "EMPLOYEE_REFERENCE", "LANDLORD_REFERENCE"] |
| newStatus | string      | One of ["NOT_STARTED", "IN_PROGRESS", "DONE"]                                                    |
| updatedAt | ISODateTime |                                                                                                  |

`instant` data structure:

| Parameter | Type   | Description                                                              |
| --------- | ------ | ------------------------------------------------------------------------ |
| status    | string | One of ["NOT_STARTED", "IN_PROGRESS", "ACCEPT", "CONSIDER", "HIGH_RISK"] |
| sections  | object | Object with sections data. See instant `sections` data structure below   |

Instant `sections` data structure:

| Parameter   | Type   | Description                                   |
| ----------- | ------ | --------------------------------------------- |
| income      | string | One of ["NOT_STARTED", "IN_PROGRESS", "DONE"] |
| rent        | string | One of ["NOT_STARTED", "IN_PROGRESS", "DONE"] |
| creditCheck | string | One of ["NOT_STARTED", "IN_PROGRESS", "DONE"] |
| savings     | string | One of ["NOT_STARTED", "IN_PROGRESS", "DONE"] |

`full` data structure:

| Parameter | Type   | Description                                                              |
| --------- | ------ | ------------------------------------------------------------------------ |
| status    | string | One of ["NOT_STARTED", "IN_PROGRESS", "ACCEPT", "CONSIDER", "HIGH_RISK"] |
| sections  | object | Object with sections data. See full `sections` data structure below      |

Full `sections` data structure:

| Parameter         | Type   | Description                                   |
| ----------------- | ------ | --------------------------------------------- |
| employerReference | string | One of ["NOT_STARTED", "IN_PROGRESS", "DONE"] |
| landlordReference | string | One of ["NOT_STARTED", "IN_PROGRESS", "DONE"] |

## Unregister Webhook

```javascript
const axios = require("axios");

const canopyEndpointBaseUri = "canopy_base_uri";
const clientId = "client_id";
const webhookType = "PASSPORT_STATUS_UPDATES";

axios({
  url: `${canopyEndpointBaseUri}/referencing-requests/client/${clientId}/webhook/${webhookType}`,
  method: "DELETE",
  headers: {
    "Authorization": "authorization_token"
    "x-api-key": "api_key",
  }
});
```

```php
<?php
$canopyEndpointBaseUri = "canopy_base_uri";
$apiKey = "your_api_key";
$authorizationToken = "authorization_token";
$clientId = "your_client_id";

$client = new \GuzzleHttp\Client();
$response = $client->request(
  "DELETE",
  "$canopyEndpointBaseUri/referencing-requests/client/$clientId/webhook/$webhookType",
  [
    "headers" => [
      "Authorization" => $authorizationToken,
      "x-api-key" => $apiKey
    ]
  ]
);

echo $response->getBody(), "\n";
?>
```

> The above command returns JSON structured like this:

```json
{
  "success": true,
  "webhookType": "PASSPORT_STATUS_UPDATES"
}
```

This endpoint unregisters webhook and stops sending Rent Passport updates to the specified callback URL.

### HTTP Request

`DELETE /referencing-requests/client/:clientId/webhook/:webhookType`

### URL Parameters

| Parameter   | Description                                    |
| ----------- | ---------------------------------------------- |
| clientId    | Your client reference                          |
| webhookType | The type of the webhook you want to unregister |
