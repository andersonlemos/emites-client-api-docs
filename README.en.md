# emites-client-api-docs

Public documentation for the Emites-Client product API.

## Introduction

Emites-Client is a Nexaas solution for issuing electronic invoices (NF-e) and consumer electronic invoices (NFC-e) developed in Java language. These documents can be issue in online or offline mode.

Integration with the application is accomplished by sending messages to a TCP/IP socket interface.

## Authentication

The authentication mechanism is under development.

## Protocol

All messages exchanged by the Emites-Client must follow the following text format:

```
Size:Identifier:Payload
```

where:

- **Size**: Numeric string representing the size of the data that follows it, also considering the separators (':'). Do not use zeros to the left;
- **Identifier**: string that identifies the type of message;
- **Payload**: string containing the payload in JSON format.

For example if we consider the following message:

```
27:TESTE:{ "teste": "teste" }
```

then we have:

- 27 is the size of the message data: 1 (**":"**) + 5 (**"TESTE"**) + 1 (**":"**) + 20 (JSON length);
- **"TESTE"** is the type of message being sent;
- after the last separator we have the JSON test payload.

Comments:
- The message should use encoding `US-ASCII` or` ISO-8859-1` (or variations). Multibyte encodings such as `UTF-8` should not be used;
- When sending a message to Emites-Client, wait for the corresponding response before sending a new request;
- The JSON payload can be sent on a single line or may contain line breaks, provided that the size of the breaks are considered in the total size of the message;
- To indicate an optional value missing from the JSON payload, simply omit the field; alternatively, it can be serialized with `null` value. Avoid sending missing values as empty strings (""), because Emites-Client performs minimum size validation if the field is not `null`;

## Creating a NFC-e

To create a NFC-e, send a message with the identifier `CREATE_NFCE`. The same identifier will be returned in the response.

Use as reference the following documents:

- JSON [request schema](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/schema/create_nfce_request_schema.json);
- JSON [response schema](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/schema/create_nfce_response_schema.json);
- [Sample JSON request](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/examples/nfce_request.json);
- [Sample JSON response](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/examples/nfce_response.json);

The response will contain the same fields that were sent in the request, and additionally the following unique fields:
- Taxes (field `produto.tributacao`, for each product);
- Note's number (field `numero`);
- Note's serie (field `serie`);
- DANFE URL (field `danfe_url`);
- XML URL (field `xml_url`);
- Base64 encoded DANFE (field `danfe`);
- Base64 encoded XML (field `xml`);

## Useful links

- [JSON schema](https://json-schema.org/): Documentation of vocabulary used for JSON schemas;
- [JSON schema validator](https://www.jsonschemavalidator.net/): allows you to validate a JSON string against its schema;
