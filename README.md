# emites-client-api-docs [[English](https://github.com/myfreecomm/emites-client-api-docs/blob/master/README.en.md)]

Documentação pública da API do produto Emites-Client.

## Introdução

O Emites-Client é uma solução Nexaas para emissão de notas fiscais eletrônicas (NF-e) e notas fiscais eletrônicas do consumidor (NFC-e) desenvolvida em linguagem Java. A emissão destes documentos pode ser feita em modo online ou offline.

A integração com a aplicação é realizada através do envio de mensagens para uma interface socket TCP/IP.

## Protocolo

Todas as mensagens trocadas pelo Emites-Client apresentam o seguinte formato texto:

```
Tamanho:Identificador:Payload
```

onde

- **Tamanho**: string númerica que representa o tamanho dos dados que a sucedem, considerando também os separadores (':'). Não utiliza zeros à esquerda;
- **Identificador**: string que identifica o tipo da mensagem;
- **Payload**: string contendo o payload em formato JSON.

Por exemplo se considerarmos a mensagem a seguir:

```
27:TESTE:{ "teste": "teste" }
```

temos:
- 27 é o tamanho dos dados da mensagem: 1 (**":"**) + 5 (**"TESTE"**) + 1 (**":"**) + 20 (tamanho do **JSON**);
- **"TESTE"** é o tipo da mensagem sendo enviada;
- após o último separador temos o payload **JSON** de teste.

Observações:
- A mensagem deve utilizar _encoding_ `US-ASCII` ou `ISO-8859-1` (ou variações). Não devem ser utilizados _encodings multibyte_ tal como `UTF-8`;
- Ao enviar uma mensagem para o Emites-Client, aguardar a resposta da mesma antes de enviar uma nova requisição;
- O payload JSON pode ser enviado em uma única linha ou pode conter quebras de linha, desde que os tamanhos das quebras sejam considerados no tamanho total da mensagem;
- Para indicar um valor opcional ausente no payload JSON, basta omitir o campo; alternativamente, ele pode ser serializado com valor `null`. Evitar enviar valores ausentes como strings vazias (""), pois o Emites-Client realiza validação de tamanho mínimo caso o campo não seja `null`;

## Criação de NFC-e

Para criar uma NFC-e (Nota Fiscal Eletrônica do Consumidor), enviar uma mensagem com o identificador `CREATE_NFCE`. O mesmo identificador será devolvido na resposta.

Utilizar como referência os documentos a seguir:

- Esquema YAML da [requisição JSON](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/schema/create_nfce_request_schema.yaml);
- Esquema YAML da [resposta JSON](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/schema/create_nfce_response_schema.yaml);
- [Exemplo de JSON de requisição](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/examples/nfce_request.json);
- [Exemplo de JSON de resposta](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/examples/nfce_response.json);

A resposta de nota aprovada conterá os mesmos campos que foram enviados na requisição, e adicionalmente os seguintes campos exclusivos:
- `status` com valor `sucesso`;
- Impostos (campo `produto.tributacao`, para cada produto);
- Número da nota (campo `numero`);
- Série da nota (campo `serie`);
- URL do DANFE (campo `danfe_url`);
- URL do XML (campo `xml_url`);
- DANFE codificado em Base64 (campo `danfe`);
- XML codificado em Base64 (campo `xml`);
- Resposta da emissão (campo `resposta_emissao`, o qual contém a chave de acesso e o número de protocolo);

### Rejeição

A resposta de nota rejeitada conterá os mesmos campos que foram enviados na requisição, e adicionalmente os seguintes campos exclusivos:
- `status` com valor `rejeitada`;
- `erros` com a lista de causas da rejeição;

O trecho a seguir mostra um exemplo de rejeição por causada por erros de validação:

```
{
  "status": "rejeitada",
  ...
  ...
  ...
  "erros": {
    "forma_de_pagamento": ["tamanho deve estar entre 1 e 100"],
    "cliente.email": ["não é um endereço de e-mail válido"]
  }
}
```

O trecho a seguir mostra um exemplo de rejeição ocorrida na SEFAZ:

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


## Cancelamento de NFC-e

Para cancelar uma NFC-e, enviar uma mensagem com o identificador `CANCEL_NFCE`. O mesmo identificador será devolvido na resposta.

O payload JSON deverá seguir o formato:

```
{ "chave_acesso": "53180922769530000131651110000001281355486170", "motivo": "Desistencia do comprador" }
```

onde:

- "chave_acesso" é a chave de acesso da NFC-e (informada na resposta de requisição de criação de NFC-e);
- "motivo" é a descrição da razão do cancelamento (opcional; se informado deve ter tamanho entre 15 e 255 caracteres)

O payload JSON da resposta do cancelamento será similar àquele devolvido durante a
operação de criação, com as seguintes diferenças [(ver exemplo)](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/examples/nfce_cancel_response.json):

- o campo `status` terá o valor `cancelada`;
- será incluído um campo adicional `cancel_xml_url` com a URL da nota cancelada;
- será incluído um campo adicional `resposta_cancelamento` (o qual contém a nova chave de acesso e novo número de protocolo);

### Rejeição

A partir de 01/10/2018 o prazo máximo para cancelamento de uma NFC-e passa a ser de 30 minutos, de acordo com o texto
do Ajuste SINIEF 07/2018. Após decorrido este prazo, uma tentativa de cancelamento da nota será respondida pela SEFAZ com
um código de rejeição (501).

Caso este cenário ocorra, a resposta devolvida pelo Emites-Client conterá no JSON o campo `erros` informando o motivo da rejeição:

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


## Criação de NF-e

Para criar uma NF-e (Nota Fiscal do Consumidor), enviar uma mensagem com o identificador `CREATE_NFE`. O mesmo identificador será devolvido na resposta.

Utilizar como referência os documentos a seguir:

- Esquema YAML da [requisição JSON](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfe/schema/create_nfe_request_schema.yaml);
- Esquema YAML da [resposta JSON](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfe/schema/create_nfe_response_schema.yaml);
- [Exemplo de JSON de requisição](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfe/examples/nfe_request.json);
- [Exemplo de JSON de resposta](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfe/examples/nfe_response.json);

A resposta de nota aprovada conterá os mesmos campos que foram enviados na requisição, e adicionalmente os seguintes campos exclusivos:
- `status` com valor `sucesso`;
- Impostos (campo `produto.tributacao`, para cada produto);
- Número da nota (campo `numero`);
- Série da nota (campo `serie`);
- URL do DANFE (campo `danfe_url`);
- URL do XML (campo `xml_url`);
- DANFE codificado em Base64 (campo `danfe`);
- XML codificado em Base64 (campo `xml`);
- Resposta da emissão (campo `resposta_emissao`, o qual contém a chave de acesso e o número de protocolo);



### Existem apenas para NFE

fields: 
  - `engine_de_calculo`
  - `contingencia`


objects: 
  - `nfe`
  - `dados_gerais`
  - `retencao_tributos`
  - `cobranca`


Obs.: Os campos de `dados_gerais` na nfe estão no raiz do nfce com o mesmo nome




## Cancelamento de NF-e

Para cancelar uma NF-e, enviar uma mensagem com o identificador `CANCEL_NFE`. O mesmo identificador será devolvido na resposta.

O payload JSON deverá seguir o formato:

```
{ "chave_acesso": "53180922769530000131651110000001281355486170", "motivo": "Desistencia do comprador" }
```

onde:

- "chave_acesso" é a chave de acesso da NFC-e (informada na resposta de requisição de criação de NFC-e);
- "motivo" é a descrição da razão do cancelamento (opcional; se informado deve ter tamanho entre 15 e 255 caracteres)

O payload JSON da resposta do cancelamento será similar àquele devolvido durante o cancelamento de NFC-e.



## Cálculo dos impostos

Para calcular os impostos para uma NFC-e (Nota Fiscal Eletrônica do Consumidor), enviar uma mensagem com o identificador `CALCULATE`. O mesmo identificador será devolvido na resposta.

Utilizar como referência os documentos usados para criação de uma NFCE-e pelo Emites Client.



## Links úteis

- [JSON schema](https://json-schema.org/): Documentação do vocabulário utilizado nos _schemas_;
- [JSON schema validator](https://www.jsonschemavalidator.net/): permite validar uma string JSON contra o respectivo _schema_;




