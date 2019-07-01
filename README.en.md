# emites-client-api-docs [[Português-BR](https://github.com/myfreecomm/emites-client-api-docs/blob/master/README.md)]

Public documentation for the Emites-Client product API.

## Introduction

Emites-Client is a Nexaas solution for issuing electronic invoices (NF-e) and consumer electronic invoices (NFC-e) developed in Java language. These documents can be issue in online or offline mode.

Integration with the application is accomplished by sending messages to a TCP/IP socket interface.

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

The response will contain the same fields that were sent in the request, and additionally the following exclusive fields:
- `status` with value equal to `sucesso`;
- Taxes (field `produto.tributacao`, for each product);
- Note's number (field `numero`);
- Note's serie (field `serie`);
- DANFE URL (field `danfe_url`);
- XML URL (field `xml_url`);
- Base64 encoded DANFE (field `danfe`);
- Base64 encoded XML (field `xml`);
- Issue response (field `resposta_emissao`, which contains the access key and protocol number);

### Rejection

The rejected response will contain the same fields that were sent in the request, and additionally the following exclusive fields:
- `status` with value equal to `rejeitada`;
- `erros` with the list of rejection reasons;

The following excerpt shows an example of rejection due to validation errors:

```
{
  "status": "rejeitada",
  ...
  ...
  ...
  "erros": {
    "forma_pagamento": ["tamanho deve estar entre 1 e 100"],
    "cliente.email": ["não é um endereço de e-mail válido"]
  }
}
```

The following excerpt shows an example of rejection raised by SEFAZ:

```
{
  "status": "rejeitada",
  ...
  ...
  ...
  "erros": {
    "rejeicao": ["471 - Rejeição: Informado NCM=00 indevidamente"]
  }
}
```


## NFC-e cancellation

To cancel an NFC-e, send a message with the identifier `CANCEL_NFCE`. The same identifier will be returned in the response.

The JSON payload should follow the format:

```
{ "chave_acesso": "53180922769530000131651110000001281355486170", "motivo": "Desistencia do comprador" }
```

where:

- `chave_acesso` is the NFC-e's access key (obtained in the creation response);
- `reason` is the description of the reason for the cancellation (optional; if informed should be between 15 and 255 characters in size)

The cancellation response's JSON payload will be similar to the one returned during the
creation operation, with the following differences [(see example)] (https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/examples/nfce_cancel_response.json):

- `status` field will have the value `cancelada`;
- an additional `cancel_xml_url` field with the updated document's URL will be included;
- an additional field `resposta_cancelamento` (which contains the new access key and new protocol number) will be included;

### Rejection

As of 10/1/2018 the maximum period to cancel a NFC-e will be 30 minutes, according to the text of the Ajuste SINIEF 07/2018. After this period elapses, an attempt to cancel the document will be answered by SEFAZ with a rejection code (501).

If this scenario occurs, the response returned by the Emites-Client will contain the field `errors` informing the reason for the rejection:

```
{
  "status": "cancelamento_rejeitado",
  ...
  ...
  ...
  "erros": {
    "rejeicao": ["501 - Rejeicao: Prazo de Cancelamento Superior ao Previsto na Legislacao"]
  }
}
```

## Useful links

- [JSON schema](https://json-schema.org/): Documentation of vocabulary used for JSON schemas;
- [JSON schema validator](https://www.jsonschemavalidator.net/): allows you to validate a JSON string against its schema;
