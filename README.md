![Logo](/canopylogo.png "Logo")

# API Documentation

## Introduction

The Canopy API is REST based.

https://en.wikipedia.org/wiki/Representational_state_transfer

We follow standard behaviour in terms of URL's, JSON request/response bodies where applicable and standard HTTP error codes.

## Environment Details

Canopy has three environments. Each environment has its own URL, which will be used by external clients in order to test functionality and then use
it on production:

- Development
- Staging
- Production

## Requesting Your Credentials

Credentials are provided on request by Canopy to yourselves. Please speak to your account manager here to obtain the details for the environments.

## Requesting an API Token

You need to request an API token to make calls to the Canopy API. In order to do this you need to:

1. Generate a payload using the clientId from the credentials you have been sent. The example here uses Javascript:

   ```
   function generatePayload(clientId) {
      const now = Math.floor(Date.now() / 1000); // sec
        const expires = now + 60 * 60;
        var payload = {
            iss: 'canopy.rent',
            scope: 'request.write_only document.read_only',
            aud: `referencing-requests/client/${clientId}/token`,
            exp: expires,
            iat: now
        };
      return payload;
    }
   ```

2. You need to sign this with JWT (e.g. jsonwebtoken in Javascript) using the secretKey in the credentials you were sent

   ```
   const jwtKey = jwt.sign(generatePayload(config), secretKey);
   ```

   The BODY of the request should include the following

   ```
   jwtKey: jwtKey
   ```

3. The header of the request should include the apiKey from the credentials you were sent

   ```
   x-api-key: apiKey
   ```

4. Finally make a POST request with the header and body to the following endpoint:

   ```
   POST /referencing-requests/client/:clientId/token
   ```

   If the request is successful, the response body will contain the token for future API requests with an expires timestamp:

   ```
   ...
   Response: {
     "success": true,
     "access_token": "Bearer eyaasd456FFGDFGdfgdfgdfgdfg7sdyfg35htjl3bhef89y4rjkbergv-KAj-dGh0xEuZftO_Utm6dugKQ",
     "expires": 1594907203
   }
   ...
   ```

## Using the Authorization Token

You need to include the x-api-key and authorization token in the headers with all Canopy API endpoint requests.

```
x-api-key: key
Authorization: token
```

This token has an expiryTime of 20 minutes, after which you will receive an authorization error, and you will then need to retrieve a new token.

## Refresh Secret Key

Requires authorization. This endpoint may be called to change current `secretKey` and get back freshly generated one. Accepts currently active `secretKey` in request body. After receiving successful response with new `secretKey`, previous `secretKey` becomes stale and should not be used for generation `jwtKey`.

```
POST /referencing-requests/client/refresh
```

Request body:

```
secretKey: string;
```

Response body:

```
secretKey: string;
```

## Prefill User's Rent Passport with Existing Data

If you already have some user information, such as identity, income and (or) rent, you can send it Canopy, so we will prefill the user's Rent Passport with provided data.

### Set Identity

```
POST /referencing-requests/client-user-data/set-identity
```

Request Body

```
email: string (required)
firstName: string (required)
middleName: string (optional)
lastName: string (required)
dateOfBirth: string (required, date format YYYY-MM-DD)
phone: string (required)
addresses: [ /* an array of addresses, each of them has the following format */
  {
    startDate: string (required)
    flat: string (optional)
    houseNumber: string (required if houseName is not present)
    houseName: string (required of houseNumber is not present)
    street: string (required)
    countryCode: enum (required) - see the list of possible values below
    county: string (optional)
    town: string (required)
    postCode: string (required),
    line1: string (optional)
    line2: string (optional)
    line3: string (optional)
  }
]
```

<details>
  <summary>Click to expand the list of values for the <code>countryCode</code> field in the above request body
  </summary>
  <br>
  <p>Below you can see the list of countries which Canopy can handle. Each item has 2 fields: <code>value</code> and <code>label</code>.
  You should pass only <code>value</code> parameter to the <code>countryCode</code> field from the above request body.</p>

  <p>For example, if the country is United Kingdom, then <code>countryCode</code> field should take "GB" value, i.e <code>countryCode: "GB"</code>.
  </p>
  <pre>
{value: "GB", label: "United Kingdom"},
{value: "AF", label: "Afghanistan"},
{value: "AX", label: "Åland Islands"},
{value: "AL", label: "Albania"},
{value: "DZ", label: "Algeria"},
{value: "AS", label: "American Samoa"},
{value: "AD", label: "Andorra"},
{value: "AO", label: "Angola"},
{value: "AI", label: "Anguilla"},
{value: "AQ", label: "Antarctica"},
{value: "AG", label: "Antigua and Barbuda"},
{value: "AR", label: "Argentina"},
{value: "AM", label: "Armenia"},
{value: "AW", label: "Aruba"},
{value: "AU", label: "Australia"},
{value: "AT", label: "Austria"},
{value: "AZ", label: "Azerbaijan"},
{value: "BS", label: "Bahamas"},
{value: "BH", label: "Bahrain"},
{value: "BD", label: "Bangladesh"},
{value: "BB", label: "Barbados"},
{value: "BY", label: "Belarus"},
{value: "BE", label: "Belgium"},
{value: "BZ", label: "Belize"},
{value: "BJ", label: "Benin"},
{value: "BM", label: "Bermuda"},
{value: "BT", label: "Bhutan"},
{value: "BO", label: "Bolivia, Plurinational State of"},
{value: "BQ", label: "Bonaire, Sint Eustatius and Saba"},
{value: "BA", label: "Bosnia and Herzegovina"},
{value: "BW", label: "Botswana"},
{value: "BV", label: "Bouvet Island"},
{value: "BR", label: "Brazil"},
{value: "IO", label: "British Indian Ocean Territory"},
{value: "BN", label: "Brunei Darussalam"},
{value: "BG", label: "Bulgaria"},
{value: "BF", label: "Burkina Faso"},
{value: "BI", label: "Burundi"},
{value: "KH", label: "Cambodia"},
{value: "CM", label: "Cameroon"},
{value: "CA", label: "Canada"},
{value: "CV", label: "Cape Verde"},
{value: "KY", label: "Cayman Islands"},
{value: "CF", label: "Central African Republic"},
{value: "TD", label: "Chad"},
{value: "CL", label: "Chile"},
{value: "CN", label: "China"},
{value: "CX", label: "Christmas Island"},
{value: "CC", label: "Cocos (Keeling) Islands"},
{value: "CO", label: "Colombia"},
{value: "KM", label: "Comoros"},
{value: "CG", label: "Congo"},
{value: "CD", label: "Congo, the Democratic Republic of the"},
{value: "CK", label: "Cook Islands"},
{value: "CR", label: "Costa Rica"},
{value: "CI", label: "Côte d'Ivoire"},
{value: "HR", label: "Croatia"},
{value: "CU", label: "Cuba"},
{value: "CW", label: "Curaçao"},
{value: "CY", label: "Cyprus"},
{value: "CZ", label: "Czech Republic"},
{value: "DK", label: "Denmark"},
{value: "DJ", label: "Djibouti"},
{value: "DM", label: "Dominica"},
{value: "DO", label: "Dominican Republic"},
{value: "EC", label: "Ecuador"},
{value: "EG", label: "Egypt"},
{value: "SV", label: "El Salvador"},
{value: "GQ", label: "Equatorial Guinea"},
{value: "ER", label: "Eritrea"},
{value: "EE", label: "Estonia"},
{value: "ET", label: "Ethiopia"},
{value: "FK", label: "Falkland Islands (Malvinas)"},
{value: "FO", label: "Faroe Islands"},
{value: "FJ", label: "Fiji"},
{value: "FI", label: "Finland"},
{value: "FR", label: "France"},
{value: "GF", label: "French Guiana"},
{value: "PF", label: "French Polynesia"},
{value: "TF", label: "French Southern Territories"},
{value: "GA", label: "Gabon"},
{value: "GM", label: "Gambia"},
{value: "GE", label: "Georgia"},
{value: "DE", label: "Germany"},
{value: "GH", label: "Ghana"},
{value: "GI", label: "Gibraltar"},
{value: "GR", label: "Greece"},
{value: "GL", label: "Greenland"},
{value: "GD", label: "Grenada"},
{value: "GP", label: "Guadeloupe"},
{value: "GU", label: "Guam"},
{value: "GT", label: "Guatemala"},
{value: "GG", label: "Guernsey"},
{value: "GN", label: "Guinea"},
{value: "GW", label: "Guinea-Bissau"},
{value: "GY", label: "Guyana"},
{value: "HT", label: "Haiti"},
{value: "HM", label: "Heard Island and McDonald Islands"},
{value: "VA", label: "Holy See (Vatican City State)"},
{value: "HN", label: "Honduras"},
{value: "HK", label: "Hong Kong"},
{value: "HU", label: "Hungary"},
{value: "IS", label: "Iceland"},
{value: "IN", label: "India"},
{value: "ID", label: "Indonesia"},
{value: "IR", label: "Iran, Islamic Republic of"},
{value: "IQ", label: "Iraq"},
{value: "IE", label: "Ireland"},
{value: "IM", label: "Isle of Man"},
{value: "IL", label: "Israel"},
{value: "IT", label: "Italy"},
{value: "JM", label: "Jamaica"},
{value: "JP", label: "Japan"},
{value: "JE", label: "Jersey"},
{value: "JO", label: "Jordan"},
{value: "KZ", label: "Kazakhstan"},
{value: "KE", label: "Kenya"},
{value: "KI", label: "Kiribati"},
{value: "KP", label: "Korea, Democratic People's Republic of"},
{value: "KR", label: "Korea, Republic of"},
{value: "KW", label: "Kuwait"},
{value: "KG", label: "Kyrgyzstan"},
{value: "LA", label: "Lao People's Democratic Republic"},
{value: "LV", label: "Latvia"},
{value: "LB", label: "Lebanon"},
{value: "LS", label: "Lesotho"},
{value: "LR", label: "Liberia"},
{value: "LY", label: "Libya"},
{value: "LI", label: "Liechtenstein"},
{value: "LT", label: "Lithuania"},
{value: "LU", label: "Luxembourg"},
{value: "MO", label: "Macao"},
{value: "MK", label: "Macedonia, the former Yugoslav Republic of"},
{value: "MG", label: "Madagascar"},
{value: "MW", label: "Malawi"},
{value: "MY", label: "Malaysia"},
{value: "MV", label: "Maldives"},
{value: "ML", label: "Mali"},
{value: "MT", label: "Malta"},
{value: "MH", label: "Marshall Islands"},
{value: "MQ", label: "Martinique"},
{value: "MR", label: "Mauritania"},
{value: "MU", label: "Mauritius"},
{value: "YT", label: "Mayotte"},
{value: "MX", label: "Mexico"},
{value: "FM", label: "Micronesia, Federated States of"},
{value: "MD", label: "Moldova, Republic of"},
{value: "MC", label: "Monaco"},
{value: "MN", label: "Mongolia"},
{value: "ME", label: "Montenegro"},
{value: "MS", label: "Montserrat"},
{value: "MA", label: "Morocco"},
{value: "MZ", label: "Mozambique"},
{value: "MM", label: "Myanmar"},
{value: "NA", label: "Namibia"},
{value: "NR", label: "Nauru"},
{value: "NP", label: "Nepal"},
{value: "NL", label: "Netherlands"},
{value: "NC", label: "New Caledonia"},
{value: "NZ", label: "New Zealand"},
{value: "NI", label: "Nicaragua"},
{value: "NE", label: "Niger"},
{value: "NG", label: "Nigeria"},
{value: "NU", label: "Niue"},
{value: "NF", label: "Norfolk Island"},
{value: "MP", label: "Northern Mariana Islands"},
{value: "NO", label: "Norway"},
{value: "OM", label: "Oman"},
{value: "PK", label: "Pakistan"},
{value: "PW", label: "Palau"},
{value: "PS", label: "Palestinian Territory, Occupied"},
{value: "PA", label: "Panama"},
{value: "PG", label: "Papua New Guinea"},
{value: "PY", label: "Paraguay"},
{value: "PE", label: "Peru"},
{value: "PH", label: "Philippines"},
{value: "PN", label: "Pitcairn"},
{value: "PL", label: "Poland"},
{value: "PT", label: "Portugal"},
{value: "PR", label: "Puerto Rico"},
{value: "QA", label: "Qatar"},
{value: "RE", label: "Réunion"},
{value: "RO", label: "Romania"},
{value: "RU", label: "Russian Federation"},
{value: "RW", label: "Rwanda"},
{value: "BL", label: "Saint Barthélemy"},
{value: "SH", label: "Saint Helena, Ascension and Tristan da Cunha"},
{value: "KN", label: "Saint Kitts and Nevis"},
{value: "LC", label: "Saint Lucia"},
{value: "MF", label: "Saint Martin (French part)"},
{value: "PM", label: "Saint Pierre and Miquelon"},
{value: "VC", label: "Saint Vincent and the Grenadines"},
{value: "WS", label: "Samoa"},
{value: "SM", label: "San Marino"},
{value: "ST", label: "Sao Tome and Principe"},
{value: "SA", label: "Saudi Arabia"},
{value: "SN", label: "Senegal"},
{value: "RS", label: "Serbia"},
{value: "SC", label: "Seychelles"},
{value: "SL", label: "Sierra Leone"},
{value: "SG", label: "Singapore"},
{value: "SX", label: "Sint Maarten (Dutch part)"},
{value: "SK", label: "Slovakia"},
{value: "SI", label: "Slovenia"},
{value: "SB", label: "Solomon Islands"},
{value: "SO", label: "Somalia"},
{value: "ZA", label: "South Africa"},
{value: "GS", label: "South Georgia and the South Sandwich Islands"},
{value: "SS", label: "South Sudan"},
{value: "ES", label: "Spain"},
{value: "LK", label: "Sri Lanka"},
{value: "SD", label: "Sudan"},
{value: "SR", label: "Suriname"},
{value: "SJ", label: "Svalbard and Jan Mayen"},
{value: "SZ", label: "Swaziland"},
{value: "SE", label: "Sweden"},
{value: "CH", label: "Switzerland"},
{value: "SY", label: "Syrian Arab Republic"},
{value: "TW", label: "Taiwan, Province of China"},
{value: "TJ", label: "Tajikistan"},
{value: "TZ", label: "Tanzania, United Republic of"},
{value: "TH", label: "Thailand"},
{value: "TL", label: "Timor-Leste"},
{value: "TG", label: "Togo"},
{value: "TK", label: "Tokelau"},
{value: "TO", label: "Tonga"},
{value: "TT", label: "Trinidad and Tobago"},
{value: "TN", label: "Tunisia"},
{value: "TR", label: "Turkey"},
{value: "TM", label: "Turkmenistan"},
{value: "TC", label: "Turks and Caicos Islands"},
{value: "TV", label: "Tuvalu"},
{value: "UG", label: "Uganda"},
{value: "UA", label: "Ukraine"},
{value: "AE", label: "United Arab Emirates"},
{value: "US", label: "United States"},
{value: "UM", label: "United States Minor Outlying Islands"},
{value: "UY", label: "Uruguay"},
{value: "UZ", label: "Uzbekistan"},
{value: "VU", label: "Vanuatu"},
{value: "VE", label: "Venezuela, Bolivarian Republic of"},
{value: "VN", label: "Vietnam"},
{value: "VG", label: "Virgin Islands, British"},
{value: "VI", label: "Virgin Islands, U.S."},
{value: "WF", label: "Wallis and Futuna"},
{value: "EH", label: "Western Sahara"},
{value: "YE", label: "Yemen"},
{value: "ZM", label: "Zambia"},
{value: "ZW", label: "Zimbabwe"}
  </pre>
</details>

Response Body

```
email: string;
firstName: string;
lastName: string;
dateOfBirth: ISODate;
phone: string;
addresses - array of provided addresses;
middleName: string (optional);
```

### Set Income

```
POST /referencing-requests/client-user-data/set-income
```

Request Body

```
email: string, (required);
data: { /* required field with the following structure: */
  income: required array of income objects, minimum 1 item;
}
```

The objects inside the "income" array from the above request body can be any of the following:

- "EMPLOYED" type

  ```
  incomeSource: "EMPLOYED", required
  annualSalary: integer, required
  paymentFrequency: string, one of ["MONTHLY", "TWO_WEEKLY", "WEEKLY"], required
  additionalInfo: string, optional
  employment: {
    companyName: string, required
    jobTitle: string, required
    employmentStatus: string, one of ["FULL_TIME", "PART_TIME"], required
    employmentBasis: string, one of ["PERMANENT", "CONTRACT"], required
    startDate: string, date format YYYY-MM-DD, required
  }
  ```

- "SELF_EMPLOYED" type

  ```
  incomeSource: "SELF_EMPLOYED", required
  annualSalary: integer, required
  paymentFrequency: string, one of ["MONTHLY", "TWO_WEEKLY", "WEEKLY"], required
  additionalInfo: string, optional,
  employment: {
    incomeName: string, required
    jobTitle: string, required
    startDate: string, date format YYYY-MM-DD, required
  }
  ```

- "STUDENT" type

  ```
    incomeSource: "STUDENT", required
    annualSalary: integer, required
    paymentFrequency: string, one of ["MONTHLY", "TWO_WEEKLY", "WEEKLY", "OTHER"], required
    additionalInfo: string, optional,
    employment: {
      educationalInstitutionName: string, required
      grantsAvailability: boolean, required
      startDate: string, date format YYYY-MM-DD, required if grantsAvailability is true
    }
  ```

- "RETIRED", "UNEMPLOYED", "BENEFITS" or "OTHER" type
  ```
    incomeSource: string, ["RETIRED", "UNEMPLOYED", "BENEFITS", "OTHER"], required
    annualSalary: integer, required
    paymentFrequency: string, ["MONTHLY", "TWO_WEEKLY", "WEEKLY", "OTHER"], required
    additionalInfo: string, optional,
    employment: {
      incomeName: string, required
      description: string, required
      startDate: string, date format YYYY-MM-DD, required
    }
  ```

Response Body

```
clientId: uuid;
email: string;
income: [ /* array of objects, each of them has the following structure */
  {
    incomeSource: string, one of ["EMPLOYED", "SELF_EMPLOYED", "STUDENT", "RETIRED", "UNEMPLOYED", "BENEFITS", "OTHER"];
    annualSalary: integer;
    paymentFrequency: string, one of ["MONTHLY", "TWO_WEEKLY", "WEEKLY", "OTHER"];
    additionalInfo: string, optional;
    employment: object, can take one of the schemas below;
  }
]
```

The "employment" field from the above response can be one of the following:

- ```
  Employed {
    companyName: string;
    jobTitle: string;
    employmentStatus: string, one of ["FULL_TIME", "PART_TIME"];
    employmentBasis: string, one of ["PERMANENT", "CONTRACT"];
    startDate: Date;
  }
  ```

- ```
  SelfEmployed {
    incomeName: string;
    jobTitle: string;
    startDate: Date;
  }
  ```

- ```
  Student {
    educationalInstitutionName: string;
    grantsAvailability: boolean;
    startDate?: Date;
  }
  ```

- ```
  Retired {
    incomeName: string;
    description: string;
    startDate: Date;
  }
  ```

- ```
  Unemployed {
    incomeName: string;
    description: string;
    startDate: Date;
  }
  ```

- ```
  Benefits {
    incomeName: string;
    description: string;
    startDate: Date;
  }
  ```

- ```
  Other {
    incomeName: string;
    description: string;
    startDate: Date;
  }
  ```

### Set Rent

```
POST /referencing-requests/client-user-data/set-rent
```

Request Body

```
email: string, required,
data: {
  homeowner: boolean, required
  rentsDuringLastYear: boolean, required
  rents: [ required if rentsDuringLastYear is true, minimum 1 item
    {
      name: string, required
      paymentFrequency: string, one of ["MONTHLY", "TWO_WEEKLY", "WEEKLY", "OTHER"], required
      rentPaymentAmount: integer, required
      rentPaidTo: string, required
    }
  ]
}
```

Response Body

```
clientId: uuid;
email: string;
rentData: {
  homeowner: boolean;
  rentsDuringLastYear: boolean;
  rents: [ /* optional field */
    {
      name: string;
      paymentFrequency: string, one of ["MONTHLY", "TWO_WEEKLY", "WEEKLY", "OTHER"];
      rentPaymentAmount: integer;
      rentPaidTo: string;
    }
  ];
```

## Referencing Endpoints

Inside the Canopy system, there are 3 main entities: company, branch and agent. All communications between the agents and renters happen on the branch level, not on the company level. It means that each company must have at least one branch (let’s call it _default_), so all existing or new agents should belong to the particular branch(es) in order to work with the potential or actual renters. For example, if the agent belongs to **Branch 1** and this agent sends the invite to the potential renter, actually the invite is sent from **Branch 1**, not directly from the agent or **Company**.

<p align="center"><img src="/company-branch.svg" /></p>

Therefore, in order to work with us, we create a Company and the default Branch for you at the very beginning. On the basic level, it’s enough for the correct workflow. There are the following possible scenarios:

- **You work on the Company level and don’t have such concept as Branch**:
  </br> In that case, there is no need to Link your branches with Canopy branches. For all the internal interactions, we will use the default branch (**Branch 1**) created for you.
- **Beside the Company, you have one Branch as well so you work on the Branch level as we do**:
  </br> In that case, you can link your single branch with our default branch, but it’s also an optional step. Anyway, we will use the default **Branch 1** for our internal communication with the renters.
- **Beside the Company, you have several branches so you work on the Branch level as we do:**
  </br>This is basically the same approach which is used inside the Canopy system. In order to request a reference for the particular renter, you can either:

  - Use the default branch, which is already created in the Canopy
  - Preliminary create an additional branches (Branch 2, Branch 3, …, Branch N) in Canopy system and then link/map them with your own branches.

  After that, at the moment of requesting a reference from Canopy for the user, you can specify the branch to which this reference should be attached. But again, this step of branch mapping is an optional step. If you don’t want to link anything, just skip the Branch Linking step so that we will always use the default branch for the internal interactions with renter.

### Get the List of Branches and Connections

The endpoint below returns the list of canopy branches associated with `clientId` and connections with client branches. Canopy branch might be linked with some client branch, in this case `clientBranchId` value will contain id of the client branch. If canopy branch linked with multiple client branches, then same canopy branch will appear in the list multiple times with different `clientBranchId`. If canopy branch is not linked to any client branch, then `clientBranchId` will be equal to `null`.

```
GET /referencing-requests/client/:clientId/branches-list
```

Parameters:

```
clientId: your client reference
```

Response body:

```
/* an array of branches, each of them has the following structure */

"branches": [
  {
    "canopyBranchId": string — id of the Canopy branch,
    "clientBranchId": string or null — id of the client's branch that is linked with this canopy branch, if equals null than this canopy branch is not linked with any client branch
    "branchName": string — the name of the Canopy branch,
    "branchAddress": { - branch address entity
      "id": string, - id of the branch address
      "line1": string,
      "line2": string,
      "line3": string,
      "street": string,
      "town": string,
      "postCode": string,
      "countryCode": string,
      "flat": string,
      "district": string,
      "houseNumber": string,
      "houseName": string
    },
  }
]
```

### Link Branch

The endpoint below links existing Canopy branch with client's branch. This operation is required for [referencing request](https://github.com/insurestreetltd/canopy-apidocs/tree/master#request-referencing). `clientBranchId` should be unique, so there can be only one connection with specific client branch. `canopyBranchId` can be used in multiple connections, so there can be multiple connections between single canopy branch and multiple client's branches.

```
POST /referencing-requests/client/:clientId/link-branch
```

Parameters:

```
clientId: your client reference
```

Request body:

```
canopyBranchId: string - id of the Canopy branch, you can get list of possible branches using GET /bracnhes-list endpoint
clientBranchId: string - id of the branch in the client's system, it can be any string of uuid format and should be unique
```

Successful response body:

```
"canopyBranchId": string — id of the Canopy branch
"clientBranchId": string — id of the branch in the client's system
"branchName": string — name of the Canopy branch
"branchAddress": { - branch address entity
  "id": string,
  "line1": string,
  "line2": string,
  "line3": string,
  "street": string,
  "town": string,
  "postCode": string,
  "countryCode": string,
  "flat": string,
  "district": string,
  "houseNumber": string,
  "houseName": string
}
```

Unsuccessful response body:

```
/* If requested Canopy branch is not associated with client's agency you’ll get the following error */

"success": false,
"requestId" — the identifier of the request,
"error": {
  "status": 400,
  "type" — error type,
  "message" — “Requested canopy branch is not associated with the client's agency”,
}
```

### Delete Branch Mapping

The endpoints below deletes existing branch mapping between Canopy branch and client branch:

```
DELETE /referencing-requests/client/:clientId/branch-mapping/:clientBranchId
```

Parameters:

```
clientId: your client reference
clientBranchId: id of the branch linked with the Canopy branch
```

Response body:

```
clientBranchId: string
```

### Request Referencing

This endpoint creates new referencing request.

```
POST /referencing-requests/client/:clientId/request
```

Parameters:

```
clientId: your client reference
```

Request body:

```
email: string (required),
firstName: string (optional),
lastName: string (optional),
callbackUrl: string (required) - URL to which Canopy will send PDF Report,
requestType: enum (required) - one of [RENTER_SCREENING, GUARANTOR_SCREENING],
itemType: enum (required) - one of [INSTANT, FULL],
title: string (optional) - it's a title used before a surname or full name,
phone: string (optional),
branchId: string (optional) - clientBranchId that is connected to any canopy branch, you can get list of created connections using GET /bracnhes-list endpoint, if there's no connection created with such branchId then this API call will return 404 error, new connection can be created using POST /link-branch endpoint, if no branchId is passed than default branch will be used for the referencing request
clientReferenceId: string (optional) - this is unique identifier on the client's side
isGuarantorNeeded: boolean (optional) - Flag allowing to request a guarantor for a specified user (user email in request). Applicable only if requestType = "RENTER_SCREENING"
```

If a referencing request is registered successfully you will receive the following response:

```
"requestId" - the identifier of the request,
"success": true
"canopyReferenceId"
```

If there was an `authentication` error while handling a new request, then you will receive the following:

```
"success": false,
"requestId" - the identifier of the request,
"error": {
  "status": 401,
  "type" - error type,
  "message" - error message
}
```

If there was a `validation` error while handling a new request, then you will receive the following:

```
"success": false,
"requestId" - the identifier of the request,
"error": {
  "status": 400,
  "type" - error type,
  "message" - error message,
  "errors": [] - an array of error objects, each of them has the following structure:
    {
      "type" - error type,
      "path" - error path,
      "message" - error message
    }
}
```

### Rent Passport Retrieval

Once referencing has been completed by the renter in the Canopy mobile application, an update message will be sent to the `callbackUrl` provided at the moment of referencing request. This message will have the following structure:

```
clientReferenceId: string,
canopyReferenceId: uuid,
document: {
  documentType: enum - one of [0, 1]; /* 0 means INSTANT screening type, 1 means FULL screening type */
  url: `/referencing-requests/client/:clientId/documents/:documentId`,
  maxRent: number,
  status: string,
  title: enum - one of [INSTANT, FULL],
  isGuarantorReference: boolean, /* exists if requestType of reference request was GUARANTOR_SCREENING
  isGuarantorNeeded: boolean, /* exists if requestType of reference request was RENTER_SCREENING
  waitingForGuarantor?: boolean, /* exists if isGuarantorNeeded has true value
  guarantorCompleteRP?: boolean /* exists if isGuarantorNeeded has true value
}
```

- If `requestType` of reference request is `GUARANTOR_SCREENING`, after the user completes his referencing, the Report will be generated as for the Guarantor.
- If `isGuarantorNeeded` of reference request is `true`, we will send 1 or 2 Reports:
  - First case (sending only one Report): Renter and guarantor have completed their Referencings - we send a Report with both data (renter and guarantor).
  - Second case (sending two Reports): Renter has completed Referencing, but the guarantor is still not present - we send the Report only with the user's data. After the Guarantor completes the referencing, we will send one more Report with both data (renter and guarantor).

```

```

To retrieve a PDF Rent Passport after successful referencing completion, you can use the `url` field from the above response:

```
GET /referencing-requests/client/:clientId/documents/:documentId
```

Parameters:

```
clientId: your client reference
documentId: the ID of the document returned from the document update
```

Response:

The PDF will be included in the response body with the filename specified in the header as follows:

```
content-type: application/pdf
content-disposition: attachment; filename='9e6222ee-312b-2297-a03a-07700363a6b6_FULL.pdf'
```

### Get Intermediate PDF Report

Get Screening Results of the renter, even if Passport is not completed. When you receive notifications that some of the Renter Passports sections were updated, you can call and get a PDF Report with current state. For example it is useful when renter should provide FULL referencing, but you wish to see INSTANT screening results as soon as it is completed and not wait while full screening will be passed. If `requestType` of last reference request for user is `GUARANTOR_SCREENING`, the Report will be generated as for the Guarantor.

```
GET /referencing-requests/client/:clientId/rent-passport/:canopyReferenceId
```

Parameters:

```
clientId – your client reference,
canopyReferenceId – the id of registered request on the Canopy side,

Also, this endpoint may accept ?format=json query param to return RP in json format, in other cases returns pdf.
```

### Revoke Guarantor Request

The endpoint below revokes guarantor request for the specified user.

```
DELETE /referencing-requests/client/:clientId/revoke-guarantor-request
```

Parameters:

```
clientId: your client reference,
```

Request body:

```
email: string (required),
```

Successful response body:

```
"success": true,
```

Unsuccessful response body:

```
/* If guarantor request has already been revoked for the specified email */

"success": false,
"requestId" — the identifier of the request,
"error": {
  "status": 409,
  "type" — error type,
  "message" — “Additional option NEED_GUARANTOR has already been revoked for email: {email}”,
}
```

## Webhooks Endpoints

### Register Webhook

The endpoint below registers the webhook with the appropriate type in Canopy system. After that, Rent Passport updates will be sent to the specified callback url.

```
POST /referencing-requests/client/:clientId/webhook/register
```

Parameters:

```
clientId: your client reference
```

Request body:

```
type: string (required) - scpecifies which type of updates will be sent from Canopy, one of ["PASSPORT_STATUS_UPDATES", "REQUEST_STATUS_UPDATES"];
callbackUrl: string (required) - the URL of the webhook endpoint;
additionalSettings: string[] (optional) - list of Rent Passport sections for which updates will be sent;
  - This field should only be added in case "PASSPORT_STATUS_UPDATES" webhook type is specified.
  - The following sections can be specified inside the array: ["INCOME", "RENT", "CREDIT_CHECK", "SAVINGS", "RENTAL_PREFERENCES", "EMPLOYEE_REFERENCE", "LANDLORD_REFERENCE"]
  - There is no need to explicitly specify the names of all sections if you want to receive all updates. Just send an empty array - [].
```

Successful response body:

```
"success": true,
"webhookType": string,
"callbackUrl": string
```

If you subscribed to the `REQUEST_STATUS_UPDATES` type, the updates will be sent to the `callbackUrl` each time one of the following events trigger:

- `INVITED` - the user was invited to connect;

- `INVITE_RESENT` - resent invitation email for the user;

- `CONNECTED` - user accepts the connection;

- `CONNECTION_REJECTED` - user rejects the connection;

- `CONNECTION_STOPPED` - user stops the connection;

- `SENDING_COMPLETED_PASSPORT_FAILED` - sending the completed passport to client failed;

- `PASSPORT_COMPLETED` - user complete his passport and the document was sent. If a guarantor was requested for this user, the status means that user and guarantor complete their passports and the document with both passports was sent;

- `INVALID_APPLICATION_DETAILS` - client's request body with application details was invalid;

- `PASSPORT_COMPLETED_WAITING_FOR_GUARANTOR` - user complete his passport and the document was sent, but the guarantor requested for this user did not complete his passport;

- `GUARANTOR_REQUEST_CANCELLED` - the guarantor request for the specified user was cancelled;

Once `REQUEST_STATUS_UPDATES` event trigger, the Canopy should sent the notification to `callbackUrl` in the following format:

```
canopyReferenceId: uuid,
clientReferenceId: string,
notes: string,
```

If you are subscribed to the `PASSPORT_STATUS_UPDATES` type, the updates will be sent to the `callbackUrl` when one of the following Rent Passport sections is updated (if updates for this section requested):

- `CREDIT CHECK`

- `INCOME`

- `RENT`

- `SAVINGS`

- `LANDLORD REFERENCE` - optional, only if FULL SCREENING requested;

- `EMPLOYER REFERENCE` - optional, only if FULL SCREENING requested;

Once `PASSPORT_STATUS_UPDATES` event trigger, the Canopy should sent the notification to `callbackUrl` in the following format:

```
canopyReferenceId: uuid,
clientReferenceId: uuid,
updatedSection: {
  type: enum - one of [INCOME, RENT, CREDIT_CHECK, SAVINGS, EMPLOYEE_REFERENCE, LANDLORD_REFERENCE],
  newStatus: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
  updatedAt: ISODateTime,
},
instant: {
  status: enum - one of [NOT_STARTED, IN_PROGRESS, ACCEPT, CONSIDER, HIGH_RISK],
  sections: {
    income: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
    rent: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
    creditCheck: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
    savings: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
  };
};
full: { /* optional field, sent only if FULL SCREENING requested  */
  status: enum - one of [NOT_STARTED, IN_PROGRESS, ACCEPT, CONSIDER, HIGH_RISK],
  sections: {
    employerReference: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
    landlordReference: enum - one of [NOT_STARTED, IN_PROGRESS, DONE],
  };
};
globalStatus: enum - one of [NOT_STARTED, IN_PROGRESS, ACCEPT, CONSIDER, HIGH_RISK],
```

### Unregister Webhook

The endpoint below unregisters webhook and stops sending Rent Passport updates to the specified callback URL.

```
DELETE /referencing-requests/client/:clientId/webhook/:webhookType
```

Parameters:

```
clientId: your client reference,
webhookType: the type of the webhook you want to unregister
```

Successful response body:

```
"success": true,
"webhookType": string
```

Unsuccessful response body:

```
/* If you don't have webhook subscriptions of the specified type */

"success": false,
"requestId" — the identifier of the request,
"error": {
  "status": 404,
  "type" — error type,
  "message" — “No webhooks of the specified type found”,
}
```

## Errors

We use conventional HTTP response codes to indicate the success or failure of an API request, typically one of:

- 2xx - indicate success
- 4xx - indicate a client error with information provided (missign required parameters etc..)
- 5xx - indicate an error with the Canopy platform

We suggest you wrap 4xx errors in your client and provide a user friendly message to your end users as part of the integration.
