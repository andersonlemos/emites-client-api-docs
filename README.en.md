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

- [JSON request YAML schema](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/schema/create_nfce_request_schema.yaml);
- [JSON response YAML schema](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/schema/create_nfce_response_schema.yaml);
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


## Creating a NF-e

To create a NF-e, send a message with the identifier `CREATE_NFE`. The same identifier will be returned in the response.

Use as reference the following documents:

- [Sample JSON request](https://github.com/myfreecomm/emites-client-api-docs/blob/master/nfce/examples/nfe_request.json);
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


### Fields with different names

-----------------------------------------------------------------------
    NFE                                |  NFCE  
---------------------------------------|-------------------------------
unidade_tributaria                     |  unidade_tributavel    
                     |
quantidade_tributaria                  |  quantidade_tributavel
                                       |
"tributacao": {                        |  codigo_origem_produto
          "icms": {                    |
            "codigo_origem_produto": 2 |
          }                            |
        }                              |     
-----------------------------------------------------------------------



### Only available for NFE

fields: 
  engine_de_calculo
  contingencia

objects: 
  nfe
  dados_gerais
  retencao_tributos
  cobranca

* The `dados_gerais` fields in NFE are in the root of NFCE with the same name



## Note issuing fields

## dados_gerais (XML: ide)

Contém informações gerais e metadados sobre a NF-e. Seus atributos são:

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho         |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------|-----------------------------------------------------------
    chave_de_acesso             |   chNFe         |  Não          |     Numérico            |    44 dígitos                |   Número da chave de acesso do Documento Fiscal<br> Se não informado, a chave de acesso será gerada internamente pelo Emites
    nfe_numero                  |   nNF           |  Não          |     Numérico            |    1 a 9 dígitos             |   Número do Documento Fiscal<br> Se não informado, a numeração será gerada a partir da sequência da série cadastrada.
    uf                          |   cUF           |  Sim          |     Numérico            |    2 dígitos                 |   Código da UF do emitente do Documento Fiscal.
    codigo_mun_ocorrencia       |   cMunFG        |  Sim          |     Numérico            |    7 dígitos                 |   Código do Município de Ocorrência do Fato Gerador.
    data_saida_entrada          |   dhSaiEnt      |  Não          |     Data                |    aaaa-mm-ddThh:mm:ss-03:00 |   Data e hora de Saída ou da Entrada da Mercadoria/Produto.
    tipo_operacao               |   tpNF          |  Sim          |     Numérico            |    1 dígito                  |   Tipo de Operação, sendo 0 = Entrada e 1 = Saída.
    destino_operacao            |   idDest        |  Sim          |     Numérico            |    1 dígito                  |   Identificador de Local de destino da operação (1 - Interna; 2 - Interestadual; 3 - Exterior).
    natureza_operacao           |   natOp         |  Sim          |     Texto  e/ou número  |    1 a 60 caracteres         |   Informar a natureza da operação de que decorrer a saída ou a entrada, tais como venda, compra, transferência, devolução, importação, consignação, remessa (para fins de demonstração, de industrialização ou outra), conforme previsto na alínea 'i', inciso I, do art. 19 do Convênio s/nº de 15 de dezembro de 1970.
    codigo_natureza_operacao    |       -         |  Condicional          |     Texto               |    3 caracteres              |   <strong>* Quando existir engine de cálculo o campo será obrigatório.</strong><br><br>Código da natureza da operação utilizado das Regras de Tributação<br>001 = Compra para Comercialização;<br>002 = Venda de Mercadoria;<br>005 = Devolução de Compra de Mercadoria;<br>006 = Remessa para Conserto;<br>012 = Devolução de Venda de Mercadoria;
    indicador_consumidor_final  |   indFinal      |  Sim          |     Numérico            |    1 dígito                  |   Indica se a NF-e foi emitida para consumidor final, sendo 0 = Não e 1 = Sim.
    indicador_presenca          |   indPres       |  Sim          |     Numérico            |    1 dígito                  |   Indicador de presença do comprador no estabelecimento comercial no momento da operação. Seleção entre:<br>0 = Não se aplica (por exemplo, Nota Fiscal complementar ou de ajuste);<br>1 = Operação presencial;<br>2 = Operação não presencial, pela Internet;<br>3   = Operação não presencial, Teleatendimento;<br>4 = NFC-e em operação com entrega a domicílio;<br>9 = Operação não presencial, outros.<br>
    finalidade_nfe              |   finNFe        |  Sim          |     Numérico            |    1 dígito                  |   Finalidade de emissão da NF-e. Seleção entre:   1 - NF-e normal   2 - NF-e complementar   3 - NF-e de ajuste
    csc             |   -        |  Não          |     Numérico            |    6 dígitos                  |   Identificador do CSC <br><strong>* Somente para NFC-e. Não obrigatório caso seja informado no cadastro da organização.</strong>
    id_token               |   -        |  Não          |     Texto e/ou número            |    36 caracteres                  |   Código de Segurança do Contribuinte (antigo Token)<br><strong>* Somente para NFC-e. Não obrigatório caso seja informado no cadastro da organização.</strong>
    just_contingencia           |   xJust      |  Sim quando Contingência     |     Texto e/ou número        |    15 a 256 caracteres   | Justificativa da entrada em contingência
    data_hora_contingencia      |   dhCont     |  Sim quando Contingência     |     Data    |  aaaa-mm-ddThh:mm:ss-03:00 | Data e Hora da entrada em contingência

## cliente (XML: dest)

Contém informações sobre o destinatário da aquisição dos produtos do emitente (nota de saída) ou venda dos produtos para o emitente (nota de entrada). Seus atributos são:

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho         |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------|-----------------------------------------------------------
    cpf_cnpj                    |   CPF<br/><br/>CNPJ<br/><br/>idEstrangeiro   |  Sim          |  Numérico               |  11 dígitos<br/><br/>14 dígitos<br/><br/>5 a 20 dígitos              |  CPF ou CNPJ do destinatário, somente números. </br></br></br></br><strong>OBS:</strong> No caso de operação com o exterior, ou para comprador estrangeiro informar o número do passaporte ou outro documento legal para identificar pessoa estrangeira.
    nome                        |   xNome         |  Sim          |  Texto                  |  Até 60 caracteres           |  Razão social ou nome do destinatário
    pessoa_fisica_juridica      |   -             |  Sim          |  Texto                  |  -                           |  Campo de controle do Emites para indicar se o destinatário é pessoa física (“person”) / jurídica (“company”) / estrangeiro ("foreign").
    inscricao_estadual          |   IE            |  Não          |  Numérico               |  Até 14 dígitos              |  Informar somente os algarismos, sem ponto, hífen, barra, etc.
    inscricao_suframa           |   ISUF          |  Não          |  Numérico               |  De 8 a 9 dígitos            |  Obrigatório nas operações que se beneficiam de incentivos fiscais existentes nas áreas sob controle da SUFRAMA. A omissão desta informação impede o processamento da operação pelo Sistema de Mercadoria Nacional da SUFRAMA e a liberação da Declaração de Ingresso, prejudicando a comprovação do ingresso / internamento da mercadoria nestas áreas.
    inscricao_municipal         |   IM            |  Não          |  Texto                  |  Até 15 caracteres           |
    indicador_inscricao_estadual|   indIEDest     |  Sim          |  Numérico               |  1 dígito                    |  Indica se o destinatário é contribuinte do ICMS. Seleção entre:<br>1 = Contribuinte ICMS (informar a IE do destinatário);<br> 2  = Contribuinte isento de Inscrição no cadastro de Contribuintes do ICMS;<br>9 = Não Contribuinte, que pode ou não possuir Inscrição Estadual no Cadastro de Contribuintes do ICMS.<br><strong>Para NFC-e esse campo sempre terá o valor 9, por isso pode ser ignorado.<strong>
    email                       |   email         |  Não          |  Texto                  |  Até 60 caracteres           |
    regime_tributario_diferenciado         |   -            |  Não          |  Texto                  |  De 3 a 4 caracteres           |  Trata-se de um atributo específico para uso do engine de cálculo TaxRules da TaxWeb. <br>Preencher se a organização não tiver nenhum regime tributário diferenciado. <br><strong>Valores válidos: LFEM, LFDES e ISENLF.</strong><br><br><i>É importante que o regime tributário diferenciado (emitente) também esteja configurado na organização.</i>

### endereco (XML: enderDest)

Grupo de informações relacionadas ao endereço do destinatário. Seus atributos são:

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho         |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------|-----------------------------------------------------------
    logradouro                  |  xLgr           |  Sim          |  Texto                  |  Até 60 caracteres           |
    numero                      |  nro            |  Sim          |  Texto                  |  Até 60 caracteres           |
    complemento                 |  xCpl           |  Não          |  Texto                  |  Até 60 caracteres           |
    bairro                      |  xBairro        |  Sim          |  Texto                  |  Até 60 caracteres           |
    codigo_municipio            |  cMun           |  Sim          |  Numérico               |  7 dígitos                   |
    nome_municipio              |  xMun           |  Sim          |  Texto                  |  Até 60 caracteres           |
    cep                         |  CEP            |  Sim          |  Numérico               |  8 dígitos                   |
    uf                          |  UF             |  Sim          |  Texto                  |  2 caracteres                |
    codigo_pais                 |  cPais          |  Condicional          |  Numérico               |  4 dígitos                   | Condicional em razão do desejo do contribuinte ou do valor máximo do documento, conforme cada UF
    nome_pais                   |  xPais          |  Não          |  Texto                  |  Até 60 caracteres           |
    telefone                    |  fone           |  Não          |  Numérico               |  De 6 a 14 carateres         |

## retirada (XML: retirada)

Grupo de informações relacionadas aos campos para complementação das informações de identificação do estabelecimento:

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho         |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------|-----------------------------------------------------------
    cpf_cnpj                    | CPF<br/><br/>CNPJ<br/> | Sim    |  Numérico               |  11 dígitos<br/><br/>14 dígitos<br/>             |  CPF ou CNPJ do destinatário, somente números.
    razao_social                |  xNome          |  Não          |  Texto                  |  Até 60 caracteres           | Nome (pessoa física) ou Razão Social (Pessoa Jurídica).
    email                       |  email          |  Não          |  Texto                  |  Até 60 caracteres           |
    inscricao_estadual          |  IE             |  Não          |  Texto e/ou número      |  De 2 a 14 caracteres        |


### endereco (XML: N/A)

Grupo de informações relacionadas ao endereço do local de retirada:

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho         |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------|-----------------------------------------------------------
    logradouro                  |  xLgr           |  Sim          |  Texto                  |  Até 60 caracteres           |
    numero                      |  nro            |  Sim          |  Texto                  |  Até 60 caracteres           |
    complemento                 |  xCpl           |  Não          |  Texto                  |  Até 60 caracteres           |
    bairro                      |  xBairro        |  Sim          |  Texto                  |  Até 60 caracteres           |
    codigo_municipio            |  cMun           |  Sim          |  Numérico               |  7 dígitos                   |
    nome_municipio              |  xMun           |  Sim          |  Texto                  |  Até 60 caracteres           |
    uf                          |  UF             |  Sim          |  Texto                  |  2 caracteres                |
    cep                         |  CEP            |  Sim          |  Numérico               |  8 dígitos                   |
    codigo_pais                 |  cPais          |  Não          |  Numérico               |  4 dígitos                   |
    nome_pais                   |  xPais          |  Não          |  Texto                  |  Até 60 caracteres           |
    telefone                    |  fone           |  Não          |  Numérico               |  De 6 a 14 carateres         |

## entrega (XML: entrega)

Grupo de informações relacionadas aos campos para complementação das informações de identificação do estabelecimento:

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho         |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------|-----------------------------------------------------------
    cpf_cnpj                    | CPF<br/><br/>CNPJ<br/> | Sim    |  Numérico               |  11 dígitos<br/><br/>14 dígitos<br/>             |  CPF ou CNPJ do destinatário, somente números.
    razao_social                |  xNome          |  Não          |  Texto                  |  Até 60 caracteres           | Nome (pessoa física) ou Razão Social (Pessoa Jurídica).
    email                       |  email          |  Não          |  Texto                  |  Até 60 caracteres           |
    inscricao_estadual          |  IE             |  Não          |  Texto e/ou número      |  De 2 a 14 caracteres        |

### endereco (XML: N/A)

Grupo de informações relacionadas ao endereço do local da entrega:

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho         |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------|-----------------------------------------------------------
    logradouro                  |  xLgr           |  Sim          |  Texto                  |  Até 60 caracteres           |
    numero                      |  nro            |  Sim          |  Texto                  |  Até 60 caracteres           |
    complemento                 |  xCpl           |  Não          |  Texto                  |  Até 60 caracteres           |
    bairro                      |  xBairro        |  Sim          |  Texto                  |  Até 60 caracteres           |
    codigo_municipio            |  cMun           |  Sim          |  Numérico               |  7 dígitos                   |
    nome_municipio              |  xMun           |  Sim          |  Texto                  |  Até 60 caracteres           |
    uf                          |  UF             |  Sim          |  Texto                  |  2 caracteres                |
    cep                         |  CEP            |  Sim          |  Numérico               |  8 dígitos                   |
    codigo_pais                 |  cPais          |  Não          |  Numérico               |  4 dígitos                   |
    nome_pais                   |  xPais          |  Não          |  Texto                  |  Até 60 caracteres           |
    telefone                    |  fone           |  Não          |  Numérico               |  De 6 a 14 carateres         |

## lista_autorizacao (XML: autXML)

Conjunto de pessoas ou empresas autorizadas a obter o XML. Seus atributos são:

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho         |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------|-----------------------------------------------------------
    Número do CNPJ ou CPF       |   CNPJ          |   Sim         |     Numérico            |    14 dígitos                |    Deve ser informado o CNPJ ou CPF
                                |   CPF           |   Sim         |     Numérico            |    11 dígitos                |


## produto (XML: prod)

Contém informações sobre os produtos contidos na NF-e. No XML, o nó  prod  é subitem do nó  det  e pode conter uma ou mais ocorrência. Seus atributos são:

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho              |   Observações
--------------------------------|-----------------|---------------|-------------------------|-----------------------------------|-----------------------------------------------------------
    codigo_produto              |    cProd        |  Sim          |     Texto e/ou número   |  Até 60 caracteres                |  Codificação própria da empresa. Preencher com CFOP, caso se trate de itens não relacionados com mercadorias/produtos e se o contribuinte não possuir codificação própria. Caso preenchido com CFOP, utilizar o formato "CFOP9999".
    codigo_ean                  |    cEAN         |  Não          |     Numérico            |  8, 12, 13 ou 14 dígitos          |  Código de barras. Preencher com o código de barra GTIN-8, GTIN-12, GTIN-13 ou GTIN-14 (antigos códigos EAN, UPC e DUN- 14). Não informar este campo se o produto não possuir este código.
    descricao                   |    xProd        |  Sim          |     Texto               |  Até 120 caracteres               |  Descrição do produto.
    ncm                         |    NCM          |  Sim          |     Numérico            |  8 dígitos                        |  Nomenclatura comum do Mercosul. Itens específicos que não possuem NCM podem informar somente o código “00”.
    exncm                       |    -            |  Não          |     Numérico            |  2 dígitos                        |  Trata-se de um atributo específico para uso do engine de cálculo TaxRules da TaxWeb destinado à detalhar a característica do produto. (Ex. pauta fiscal etc).
    cest                        |    CEST         |  Não          |     Numérico            |  7 dígitos                        |  Código Especificador de Substituição Tributária.
    extipi                      |    EXTIPI       |  Não          |     Numérico            |  De 2 a 3 dígitos                 |
    cfop                        |    CFOP         |  Sim          |     Numérico            |  4 dígitos                        |  Código Fiscal de Operações e Prestações.
    produto_especifico          |         -       |       -       |            -            |          -                        |  -
    unidade_comercial           |    uCom         |  Sim          |     Texto               |  Até 6 caracteres                 |  Unidade de comercialização do produto.
    quantidade_comercial        |    qCom         |  Sim          |     Decimal             |  Até 11 dígitos, 4 casas decimais |  Quantidade de comercialização do produto.
    valor_unitario_comercial    |    vUnCom       |  Sim          |     Decimal             |  Até 11 dígitos, 4 casas decimais |  Valor unitário de comercialização.
    ean_unidade_trib            |    cEANTrib     |  Não          |     Numérico            |  8, 12, 13 ou 14 dígitos          |
    unidade_tributaria          |    uTrib        |  Sim          |     Texto               |  Até 6 caracteres                 |  Unidade tributável do produto.
    quantidade_tributaria       |    qTrib        |  Sim          |     Decimal             |  Até 11 dígitos, 4 casas decimais |  Quantidade tributável do produto.
    valor_unitario_tributario   |    vUnTrib      |  Sim          |     Decimal             |  Até 11 dígitos, 10 casas decimais|  Valor unitário de tributação.
    valor_frete                 |    vFrete       |  Não          |     Decimal             |  Até 13 dígitos, 2 casas decimais |
    valor_seguro                |  vSeg           |  Não          |     Decimal             |  Até 13 dígitos, 2 casas decimais |
    valor_desconto              |  vDesc          |  Não          |     Decimal             |  Até 13 dígitos, 2 casas decimais |
    valor_total_produto         |  vProd          |  Sim          |     Decimal             |  Até 13 dígitos, 2 casas decimais |
    outras_despesas             |  vOutro         |  Não          |     Decimal             |  Até 13 dígitos, 2 casas decimais |  Outras despesas acessórias.
    num_pedido                  |   xPed          |  Não          |     Texto e/ou número   |  Até 15 caracteres                |  Número do pedido de compra, se houver.
    num_item_pedido             |  nItemPed       |  Não          |     Numérico            |  Até 6 dígitos                    |  Item do pedido de compra.
    num_controle_fci            |   nFCI          |  Não          |     Texto               |  Até 36 caracteres                |  Número de controle da FCI - Ficha de Conteúdo de Importação.
    ind_valor_total             |  indTotal       |  Sim          |     Numérico            |  1 dígito                         |  Informa se o valor dos produtos compõem o valor total da nota. Seleção entre:<br>0 = Valor do item não compõem o valor total da NF-e;<br>1 = Valor do item compõem o valor total da NF-e.<br>
    producao_escala             |  indEscala      |  Não          |     Texto               |  1 dígito                         |  S - Produzido em Escala Relevante;<br>N – Produzido em Escala NÃO Relevante.
    cnpj_fabricante_mercadoria  |  CNPJFab        |  Não          |     Numérico            |  14 dígitos                       |  S - Produzido em Escala Relevante;<br>N – Produzido em Escala NÃO Relevante.
    codigo_beneficio_fiscal     |  cBenef         |  Não          |     Texto               |  10 dígitos                       |  Código de Benefício Fiscal utilizado pela UF, aplicado ao item.
    aplicacao     |     -     |  Não          |     Texto               |  1 dígito                       |  Aplicação do Produto (<i>Trata-se de um atributo específico para uso do engine de cálculo TaxRules da TaxWeb.</i>). Seleção entre:<br>I - Industrialização<br>C - Comercialização<br>U - Uso e Consumo<br>A - Ativo Imobilizado
    fabricacao     |     -     |  Não          |     Numérico               |  1 dígito                       |  Indica onde ocorreu a fabricação do produto (<i>Trata-se de um atributo específico para uso do engine de cálculo TaxRules da TaxWeb.</i>). Seleção entre:<br>0 - Própria<br>1 - Terceiros

## tributação (XML: imposto)

Grupo de informações relacionadas à tributação de ICMS, IPI, PIS, COFINS e Importação.

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------------|-----------------------------------------------------------
    valor_aproximado_total      |  vTotTrib     |  Sim          |     Decimal             |  Até 13 dígitos, 2 casas decimais  |  Valor aproximado total de tributos federais, estaduais e municipais para o produto.<br><i>Caso o valor esteja ausente, o Emites não enviará valor algum para a SEFAZ.</i>

### icms (XML: ICMS)

Informações relacionadas ao ICMS. Os atributos são variáveis de acordo com a situação tributária. Atributos comuns a todas as situações tributárias

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------------|-----------------------------------------------------------
situacao_tributaria             |  CST            |  Sim          |     Numérico            |    2 dígitos                       |
codigo_origem_produto           |  orig           |  Sim          |     Numérico            |    1 dígito                        |  Origem da mercadoria. Seleção entre:<br>  0 = Nacional, exceto as indicadas nos códigos 3, 4, 5 e 8;  <br>1 = Estrangeira - Importação direta, exceto a indicada no código 6;  <br>2 = Estrangeira - Adquirida no mercado interno, exceto a indicada no código 7;  <br>3 = Nacional, mercadoria ou bem com Conteúdo de Importação superior a 40% e inferior ou igual a 70%;  <br>4 = Nacional, cuja produção tenha sido feita em conformidade com os processos produtivos básicos de que tratam as legislações citadas nos Ajustes;  <br>5 = Nacional, mercadoria ou bem com Conteúdo de Importação inferior ou igual a 40%;  <br>6 = Estrangeira - Importação direta, sem similar nacional, constante em lista da CAMEX e gás natural;  <br>7 = Estrangeira - Adquirida no mercado interno, sem similar nacional, constante lista CAMEX e gás natural.  <br>8 = Nacional, mercadoria ou bem com Conteúdo de Importação superior a 70%.

### ICMS interestadual (XML: ICMSUFDest)

Grupo a ser informado nas vendas interestaduais para consumidor final, não contribuinte de ICMS.

  Campo                                 |  XML            |  Obrigatório  |  Tipo     |  Formato e tamanho                 | Observações
----------------------------------------|-----------------|---------------|-----------|------------------------------------|------------
  valor_base_calculo_uf_dest            |  vBCUFDest      |  Não          |  Decimal  | Até 13 dígitos, 2 casas decimais   |
  aliquota_interna_interestadual        |  pICMSUFDest    |  Não          |  Decimal  | Até 3 dígitos, 4 casas decimais    |
  aliquota_interestadual                |  pICMSInter     |  Não          |  Decimal  | Até 3 dígitos, 4 casas decimais    |
  perc_provisorio_interestadual         |  pICMSInterPart |  Não          |  Decimal  | Até 3 dígitos, 4 casas decimais    |
  valor_uf_remetente_interestadual      |  vICMSUFRemet   |  Não          |  Decimal  | Até 13 dígitos, 2 casas decimais   |
  valor_uf_destinatario_interestadual   |  vICMSUFDest    |  Não          |  Decimal  | Até 13 dígitos, 2 casas decimais   |
  perc_fcp_interestadual                |  pFCPUFDest     |  Não          |  Decimal  | Até 3 dígitos, 4 casas decimais    |
  valor_fcp_interestadual               |  vFCPUFDest     |  Não          |  Decimal  | Até 13 dígitos, 2 casas decimais   |

### ICMSST (XML: ICMSST) | situacao_tributaria = 41 e 60

41 = Não Tributado (v2.0);<br>
60 = cobrado anteriormente por substituição tributária (Incluído NT 2016/002);

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
base_calculo_icmsst_remetente     | vBCSTRet     |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_consumidor_final         | pST            | Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_icms_substituto             | vICMSSubstituto | Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais    | Valor do ICMS Próprio do Substituto cobrado em operação anterior
valor_icmsst_retido_remetente     | vICMSSTRet   |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_base_calculo_fcp_st_retido  | vBCFCPSTRet | Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp_st_retido            | pFCPSTRet   | Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp_st_retido               | vFCPSTRet   | Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
base_calculo_icmsst_destino       | vBCSTDest    |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_icmsst_retido_destino       | vICMSSTDest  |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
perc_reducao_base_calculo_efetivo | pRedBCEfet | Sim | Decimal | Até 3 dígitos, 4 casas decimais | Opcional a critério da UF.
valor_base_calculo_efetivo        | vBCEfet | Sim | Decimal | Até 13 dígitos, 2 casas decimais | Opcional a critério da UF.
aliquota_efetiva                  | pICMSEfet | Sim | Decimal | Até 3 dígitos, 4 casas decimais | Opcional a critério da UF.
valor_efetivo                     | vICMSEfet | Sim | Decimal | Até 13 dígitos, 2 casas decimais | Opcional a critério da UF.

### situacao_tributaria = 00

Tributada integralmente.

  Campo no XML                | XML           |  Obrigatório  |  Tipo      |  Formato e tamanho                 | Observações
------------------------------|---------------|---------------|------------|------------------------------------|------------
  modalidade_base_calculo     | modBC         |  Sim          |  Numérico  |  1 dígito                          | 0=Margem Valor Agregado (%);  <br>1=Pauta (Valor);  <br>2=Preço Tabelado Máx. (valor);  <br>3=Valor da operação.
  valor_base_calculo          | vBC           |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais  |
  aliquota_icms               | pICMS         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais  |
  valor_icms                  | vICMS         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais  |
  aliquota_fcp                | pFCP          |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais   |
  valor_fcp                   | vFCP          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais  |

### situacao_tributaria = 10

Tributada com cobrança de ICMS por ST.

  Campo                         |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
  modalidade_base_calculo       |  modBC        |  Sim          |  Numérico  |  Numérico                           | 1 dígito 0=Margem Valor Agregado (%); <br>1=Pauta (Valor); <br>2=Preço Tabelado Máx. (valor); <br>3=Valor da operação.
  valor_base_calculo            |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
  aliquota_icms                 |  pICMS        |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
  valor_icms                    |  vICMS        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
  modalidade_base_calculo_st    |  modBCST      |  Sim          |  Numérico  |  1 dígito                           | 0=Preço tabelado ou máximo sugerido; <br>1=Lista Negativa (valor); <br>2=Lista Positiva (valor); <br>3=Lista Neutra (valor);  4=Margem Valor Agregado (%); 5=Pauta (valor).
  valor_base_calculo_st         |  vBCST        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
  perc_reducao_base_calculo_st  |  pRedBCST     |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
  perc_mva_icms_st              |  pMVAST       |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
  aliquota_icms_st              |  pICMSST      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
  valor_icms_st                 |  vICMSST      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
  valor_base_calculo_fcp        | vBCFCP        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
  aliquota_fcp                  | pFCP          |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
  valor_fcp                     | vFCP          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
  valor_base_calculo_fcp_st     | vBCFCPST      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
  aliquota_fcp_st               | pFCPST        |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
  valor_fcp_st                  | vFCPST        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

#### com partilha do ICMS entre a UF de origem e a UF de destino ou a UF definida na legislação

  Campo                         |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
  base_calculo_operacao_propria |  pBCOp        |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |  Percentual para determinação do valor da Base de Cálculo da operação própria.
  uf_icmsst_devido              |  UFST         |  Sim          |  Texto     |  2 carac.                           |  Sigla da UF para qual é devido o ICMS ST da operação.

### situacao_tributaria = 20

Com redução de base de cálculo.

  Campo                         |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
modalidade_base_calculo         |  modBC        |  Sim          |  Numérico  |  1 dígito                           |  0=Margem Valor Agregado (%);<br>1=Pauta (Valor);<br>2=Preço Tabelado Máx. (valor);<br>3=Valor da operação.
valor_base_calculo              |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
perc_reducao_base_calculo       |  pRedBC       |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
aliquota_icms                   |  pICMS        |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_icms                      |  vICMS        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_icms_desonerado           |  vICMSDeson   |  Não          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
motivo_desoneracao              |  motDesICMS   |  Não          |  Numérico  |  2 dígitos                          |  Informar o motivo da desoneração: <br>1 = Táxi<br> 3 = Produto Agropecuário <br>4 = Frotista/Locadora <br>5 = Diplomático/Consular <br>6 = Utilitários e Motocicletas da Amazônia Ocidental e Áreas de Livre Comércio <br>7 = SUFRAMA <br>8 = Venda a Órgão Público <br>9 = Outros <br>10 = Deficiente Condutor <br>11 = Deficiente Não Condutor <br>12 = Órgão de fomento e desenvolvimento agropecuário
valor_base_calculo_fcp          | vBCFCP        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp                    | pFCP          |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp                       | vFCP          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

### situacao_tributaria = 30

Isenta e não tributada e com cobrança de ICMS por ST.

  Campo                         |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
valor_icms_desonerado           |  vICMSDeson   |  Não          |  Decimal   | Até 13 dígitos, 2 casas decimais    |
motivo_desoneracao              |  motDesICMS   |  Não          |  Numérico  | 2 dígitos                           |  Informar o motivo da desoneração: <br>1 = Táxi <br>3 = Produto Agropecuário <br>4 = Frotista/Locadora <br>5 = Diplomático/Consular <br>6 = Utilitários e  Motocicletas da Amazônia Ocidental e Áreas de Livre Comércio <br>7 = SUFRAMA <br>8 = Venda a Órgão Público <br>9 = Outros <br>10 = Deficiente Condutor <br>11 = Deficiente Não Condutor <br>12 = Órgão de fomento e desenvolvimento agropecuário
modalidade_base_calculo_st      |  modBCST      |  Sim          |  Numérico   | 1 dígito                           |  0=Preço tabelado ou máximo sugerido;<br>1=Lista Negativa (valor);<br>2=Lista Positiva (valor);<br>3=Lista Neutra (valor);<br>4=Margem Valor Agregado (%);<br>5=Pauta (valor).
valor_base_calculo_st           |  vBCST      |  Sim          |  Decimal   | Até 13 dígitos, 2 casas decimais    |
perc_reducao_base_calculo_st    |  pRedBCST   |  Não          |  Decimal   | Até 3 dígitos, 4 casas decimais     |
perc_mva_icms_st                |  pMVAST     |  Não          |  Decimal   | Até 3 dígitos, 4 casas decimais     |
aliquota_icms_st                |  pICMSST    |  Sim          |  Decimal   | Até 3 dígitos, 4 casas decimais     |
valor_icms_st                   |  vICMSST    |  Sim          |  Decimal   | Até 13 dígitos, 2 casas decimais    |
valor_base_calculo_fcp_st       | vBCFCPST    |  Sim          |  Decimal   | Até 13 dígitos, 2 casas decimais    |
aliquota_fcp_st                 | pFCPST      |  Sim          |  Decimal   | Até 3 dígitos, 4 casas decimais     |
valor_fcp_st                    | vFCPST      |  Sim          |  Decimal   | Até 13 dígitos, 2 casas decimais    |

### situacao_tributaria = 40, 41 e 50

40 =  Isenta
41 =  Não tributada
50 =  Suspensão

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
valor_icms_desonerado           |  vICMSDeson   |  Não          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
motivo_desoneracao              |  motDesICMS   |  Não          |  Numérico  |  2 dígitos                          |  Informar o motivo da desoneração:<br>1 = Táxi<br>3 = Produto Agropecuário<br>4 = Frotista/Locadora<br>5 = Diplomático/Consular<br>6 = Utilitários e Motocicletas da Amazônia Ocidental e Áreas de Livre Comércio<br>7 = SUFRAMA<br>8 = Venda a Órgão Público<br>9 = Outros<br>10 = Deficiente Condutor<br>11 = Deficiente Não Condutor<br>12 = Órgão de fomento e desenvolvimento agropecuário<br>

### situacao_tributaria = 41 e ICMSST devido para a UF de destino

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
base_calculo_icmsst_remetente   |  vBCSTRet     |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_icmsst_retido_remetente   |  vICMSSTRet   |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
base_calculo_icmsst_destino     |  vBCSTDest    |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_icmsst_retido_destino     |  vICMSSTDest  |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |


### situacao_tributaria = 51

Diferido.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
modalidade_base_calculo         |  modBC        |  Sim          |  Numérico  |  1 dígito                           |  0=Margem Valor Agregado (%);<br>1=Pauta (Valor);<br>2=Preço Tabelado Máx. (valor);<br>3=Valor da operação.<br>
valor_base_calculo              |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
perc_reducao_base_calculo       |  pRedBC       |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
aliquota_icms                   |  pICMS        |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_icms                      |  vICMS        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_icms_operacao             |  vICMSOp      |  Não          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |  Valor como se não tivesse o diferimento.
perc_diferimento                |  pDif         |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |  No caso de diferimento total, informar o percentual de diferimento "100".
valor_icms_diferido             |  vICMSDif     |  Não          |            |  Até 13 dígitos, 2 casas decimais   |
valor_base_calculo_fcp          | vBCFCP        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp                    | pFCP          |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp                       | vFCP          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

### situacao_tributaria = 60

Cobrado anteriormente por ST.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
base_icmsst_retido              |  vBCSTRet     |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |  Valor da Base de Cálculo do ICMS ST cobrado anteriormente por ST. O valor pode ser omitido quando a legislação não exigir a sua informação.
aliquota_consumidor_final       | pST            | Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_icms_substituto           | vICMSSubstituto | Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais    | Valor do ICMS Próprio do Substituto cobrado em operação anterior
valor_icmsst_retido             |  vICMSSTRet   |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |  Valor do ICMS ST cobrado anteriormente por ST. O valor pode ser omitido quando a legislação não exigir a sua informação.
valor_base_calculo_fcp_st_retido | vBCFCPSTRet | Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp_st_retido           | pFCPSTRet   | Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp_st_retido              | vFCPSTRet   | Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

### situacao_tributaria = 70 e 90

70 = Com redução da base de cálculo e cobrança por ST
90 = Outras

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
modalidade_base_calculo         |  modBC        |  Sim          |  Numérico  |  1 dígito                           |  0=Margem Valor Agregado (%);<br>1=Pauta (Valor);<br>2=Preço Tabelado Máx. (valor);<br>3=Valor da operação.
valor_base_calculo              |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
perc_reducao_base_calculo       |  pRedBC       |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
aliquota_icms                   |  pICMS        |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_icms                      |  vICMS        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_icms_desonerado           |  vICMSDeson   |  Não          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
motivo_desoneracao              |  motDesICMS   |  Não          |  Numérico  |  2 dígitos                          |  Informar o motivo da desoneração: <br>1 = Táxi<br>3 = Produto Agropecuário<br>4 = Frotista/Locadora<br>5 = Diplomático/Consular<br>6 = Utilitários e Motocicletas da Amazônia Ocidental e Áreas de Livre Comércio<br>7 = SUFRAMA<br>8 = Venda a Órgão Público<br>9 = Outros<br>10 = Deficiente Condutor<br>11 = Deficiente Não Condutor<br>12 = Órgão de fomento e desenvolvimento agropecuário<br>
modalidade_base_calculo_st      |  modBCST      |  Sim          |  Numérico  |  1 dígito                           |  0=Preço tabelado ou máximo sugerido;<br>1=Lista Negativa (valor);<br>2=Lista Positiva (valor);<br>3=Lista Neutra (valor);<br>4=Margem Valor Agregado (%);<br>5=Pauta (valor).<br>
valor_base_calculo_st           |  vBCST        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
perc_reducao_base_calculo_st    |  pRedBCST     |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
perc_mva_icms_st                |  pMVAST       |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
aliquota_icms_st                |  pICMSST      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_icms_st                   |  vICMSST      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_base_calculo_fcp          |  vBCFCP       |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp                    |  pFCP         |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp                       |  vFCP         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_base_calculo_fcp_st       |  vBCFCPST     |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp_st                 |  pFCPST       |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp_st                    |  vFCPST       |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

#### com partilha do ICMS entre a UF de origem e a UF de destino ou a UF definida na legislação

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
base_calculo_operacao_propria   |  pBCOp        |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |  Percentual para determinação do valor da Base de Cálculo da operação própria.
uf_icmsst_devido                |  UFST         |  Sim          |  Texto     |  2 carac.                           |  Sigla da UF para qual é devido o ICMS ST da operação.

### situacao_tributaria = 101

Tributada com permissão de crédito.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
situacao_simples_nacional       |  CSOSN        |  Sim          |  Numérico  |  3 dígitos                          |  situacao_simples_nacional
aliquota_icms_simples_nacional  |  pCredSN      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |  aliquota_icms_simples_nacional
credito_icms_simples_nacional   |  vCredICMSSN  |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |  credito_icms_simples_nacional

### situacao_tributaria = 102

Tributada sem permissão de crédito.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
situacao_simples_nacional       |  CSOSN        |   Sim         |  Numérico  |  3 dígitos                          |

### situacao_tributaria = 103

Isenção de ICMS.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
situacao_simples_nacional       |  CSOSN        |  Sim          |  Numérico  |  3 dígitos                          |

### situacao_tributaria = 201

Tributada com permissão de crédito e com cobrança de ICMS por ST.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
situacao_simples_nacional       |  CSOSN        |  Sim          |  Numérico  |  3 dígitos                          |
aliquota_icms_simples_nacional  |  pCredSN      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
credito_icms_simples_nacional   |  vCredICMSSN  |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
modalidade_base_calculo_st      |  modBCST      |  Sim          |  Numérico  |  1 dígito                           |  0=Preço tabelado ou máximo sugerido;<br>1=Lista Negativa (valor);<br>2=Lista Positiva (valor);<br>3=Lista Neutra (valor);<br>4=Margem Valor Agregado (%);<br>5=Pauta (valor).
valor_base_calculo_st           |  vBCST        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
perc_reducao_base_calculo_st    |  pRedBCST     |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
perc_mva_icms_st                |  pMVAST       |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
aliquota_icms_st                |  pICMSST      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_icms_st                   |  vICMSST      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_base_calculo_fcp_st       |  vBCFCPST     |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp_st                 |  pFCPST       |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp_st                    |  vFCPST       |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_icms_simples_nacional  |  pCredSN      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
credito_icms_simples_nacional   |  vCredICMSSN  |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

### situacao_tributaria = 202

Tributada sem permissão de crédito e com cobrança de ICMS por ST.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
situacao_simples_nacional       |  CSOSN        |  Sim          |  Numérico  |  3 dígitos
modalidade_base_calculo_st      |  modBCST      |  Sim          |  Numérico  |  1 dígito                           |  0=Preço tabelado ou máximo sugerido;<br>1=Lista Negativa (valor);<br>2=Lista Positiva (valor);<br>3=Lista Neutra (valor);<br>4=Margem Valor Agregado (%);<br>5=Pauta (valor).
valor_base_calculo_st           |  vBCST        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
perc_reducao_base_calculo_st    |  pRedBCST     |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
perc_mva_icms_st                |  pMVAST       |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
aliquota_icms_st                |  pICMSST      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_icms_st                   |  vICMSST      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
valor_base_calculo_fcp_st       |  vBCFCPST     |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp_st                 |  pFCPST       |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp_st                    |  vFCPST       |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

### situacao_tributaria = 203

Isenção de ICMS pela faixa de receita bruta e com cobrança de ICMS por ST.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
situacao_simples_nacional       |  CSOSN        |  Sim          |  Numérico  |  3 dígitos
modalidade_base_calculo_st      |  modBCST      |  Sim          |  Numérico  |  1 dígito                           |  0=Preço tabelado ou máximo sugerido;<br>1=Lista Negativa (valor);<br>2=Lista Positiva (valor);<br>3=Lista Neutra (valor);<br>4=Margem Valor Agregado (%);<br>5=Pauta (valor).
valor_base_calculo_st           |  vBCST        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
perc_reducao_base_calculo_st    |  pRedBCST     |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais
perc_mva_icms_st                |  pMVAST       |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais
aliquota_icms_st                |  pICMSST      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais
valor_icms_st                   |  vICMSST      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
valor_base_calculo_fcp_st       |  vBCFCPST     |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp_st                 |  pFCPST       |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp_st                    |  vFCPST       |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

### situacao_tributaria = 300

Imune.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
situacao_simples_nacional       |  CSOSN        |  Sim          |  Numérico  |  3 dígitos

### situacao_tributaria = 400

Não tributada

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
situacao_simples_nacional       |  CSOSN        |  Sim          |  Numérico  |  3 dígitos

### situacao_tributaria = 500

ICMS cobrado anteriormente por ST ou antecipação.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
situacao_simples_nacional       |  CSOSN        |  Sim          |  Numérico  |  3 dígitos
base_icmsst_retido              |  vBCSTRet     |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |  Valor da Base de Cálculo do ICMS ST cobrado anteriormente por ST. O valor pode ser omitido quando a legislação não exigir a sua informação.
valor_icmsst_retido             |  vICMSSTRet   |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |  Valor do ICMS ST cobrado anteriormente por ST. O valor pode ser omitido quando a legislação não exigir a sua informação.
aliquota_consumidor_final       | pST            | Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_icms_substituto           | vICMSSubstituto | Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais    | Valor do ICMS Próprio do Substituto cobrado em operação anterior
valor_base_calculo_fcp_st_retido | vBCFCPSTRet | Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp_st_retido           | pFCPSTRet   | Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp_st_retido              | vFCPSTRet   | Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

### situacao_tributaria = 900

Outros.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|------------
situacao_simples_nacional       |  CSOSN        |  Sim          |  Numérico  |  3 dígitos
aliquota_icms_simples_nacional  |  pCredSN      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais
credito_icms_simples_nacional   |  vCredICMSSN  |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
modalidade_base_calculo         |  modBC        |  Sim          |  Numérico  |  1 dígito                           |  0=Margem Valor Agregado (%);<br>1=Pauta (Valor);<br>2=Preço Tabelado Máx. (valor);<br>3=Valor da operação.
valor_base_calculo              |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
perc_reducao_base_calculo       |  pRedBC       |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais
aliquota_icms                   |  pICMS        |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais
valor_icms                      |  vICMS        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
modalidade_base_calculo_st      |  modBCST      |  Sim          |  Numérico  |  1 dígito                           |  0=Preço tabelado ou máximo sugerido;<br>1=Lista Negativa (valor);<br>2=Lista Positiva (valor);<br>3=Lista Neutra (valor);<br>4=Margem Valor Agregado (%);<br>5=Pauta (valor).
valor_base_calculo_st           |  vBCST        |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
perc_reducao_base_calculo_st    |  pRedBCST     |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais
perc_mva_icms_st                |  pMVAST       |  Não          |  Decimal   |  Até 3 dígitos, 4 casas decimais
aliquota_icms_st                |  pICMSST      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais
valor_icms_st                   |  vICMSST      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
valor_icms_operacao             |  vICMSOp      |  Não          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |  Valor como se não tivesse o diferimento.
valor_base_calculo_fcp_st       |  vBCFCPST     |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_fcp_st                 |  pFCPST       |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_fcp_st                    |  vFCPST       |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

### ipi (XML: IPI)

Atributos comuns a todas as situações tributárias

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------------|-----------------------------------------------------------
    situacao_tributaria         |  CST            |  Sim          |     Numérico            |  2 dígitos                         |
    cnpj_produtor               |  CNPJProd       |  Não          |     Numérico            |  14 dígitos                        |
    codigo_selo_controle        |  cSelo          |  Não          |     Texto               |  Até 60 caracteres                 |  Código do selo de controle IPI
    quantidade_selo_controle    |  qSelo          |  Não          |     Numérico            |  Até 12 dígitos                    |  Quantidade de selo de controle
    codigo_enquadramento        |  cEnq           |  Sim          |     Numérico            |  Até 3 dígitos                     |  Geralmente o valor informado é 999

### situacao_tributaria = 00, 49, 50 e 99

00 = Entrada com recuperação de crédito.
49 = Outras entradas
50 = Saída tributada
99 = Outra saídas

### Cálculo por alíquota

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------|------------|-------------------------------------|--------------
valor_base_calculo              |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |
aliquota_ipi                    |  pIPI         |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais    |
valor_ipi                       |  vIPI         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais   |

### Cálculo por valor

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
valor_unidade                   |  vUnid        |  Sim          |  Decimal   |  Até 11 dígitos, de 0 a 4 casas decimais |
valor_ipi                       |  vIPI         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais

### situacao_tributaria = 01

Entrada tributada com alíquota zero. Não possui campos para informar alíquota.

### situacao_tributaria = 02
Entrada Isenta. Não possui campos para informar alíquota.

### situacao_tributaria = 03
Entrada não-tributada. Não possui campos para informar alíquota.

### situacao_tributaria = 04
Entrada imune. Não possui campos para informar alíquota.

### situacao_tributaria = 05
Entrada com suspensão. Não possui campos para informar alíquota.

### situacao_tributaria = 51
Saída tributada com alíquota zero. Não possui campos para informar alíquota.

### situacao_tributaria = 52
Saída isenta. Não possui campos para informar alíquota.

### situacao_tributaria = 53
Saída não-tributada. Não possui campos para informar alíquota.

### situacao_tributaria = 54
Saída imune. Não possui campos para informar alíquota.

### situacao_tributaria = 55
Saída com suspensão. Não possui campos para informar alíquota.

### imposto_devolvido (XML: impostoDevol)

O motivo da devolução deverá ser informado pela empresa no campo de Informações Adicionais do Produto.

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------------|-----------------------------------------------------------
    perc_mercadoria_devolvida   |  pDevol         |  Sim          |     Decimal             |  Até 3 dígitos, 2 casas decimais   |
    valor_ipi_devolvido         |  vIPIDevol      |  Sim          |     Decimal             |  Até 13 dígitos, 2 casas decimais  |

### pis (XML: PIS)

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------------|-----------------------------------------------------------
    situacao_tributaria         |  CST            |  Sim          |  Numérico               |  2 dígitos                         |   Atributos comuns a todas as situações tributárias

### situacao_tributaria = 01 e 02

01 = Operação Tributável com Alíquota Básica
02 = Operação Tributável com Alíquota Diferenciada

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
valor_base_calculo              |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
aliquota_pis                    |  pPIS         |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais
valor_pis                       |  vPIS         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais

### situacao_tributaria = 03

Operação Tributável com Alíquota por Unidade de Medida de Produto.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
aliquota_pis_reais              |  vAliqProd    |  Sim          |  Decimal   |  Até 11 dígitos, de 0 a 4 casas decimais
quantidade_vendida              |  qBCProd      |  Sim          |  Decimal   |  Até 12 dígitos, de 0 a 4 casas decimais
valor_pis                       |  vPIS         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais

### situacao_tributaria = 04, 06, 07, 08, 09

04 = Operação Tributável Monofásica - Revenda a Alíquota Zero
06 = Operação Tributável a Alíquota Zero
07 = Operação Isenta da Contribuição
08 = Operação sem Incidência da Contribuição
09 = Operação com Suspensão da Contribuição

Não há campos para informar alíquota.

### situacao_tributaria = 05

Operação Tributável por Substituição Tributária.

### Cálculo por percentual

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
valor_base_calculo_st           |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
aliquota_st                     |  pPIS         |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais
valor_pis_st                    |  vPIS         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais

### Cálculo por valor

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
aliquota_pis_reais_st           |  vAliqProd    |  Sim          |  Decimal   |  Até 11 dígitos, de 0 a 4 casas decimais
quantidade_vendida_st           |  qBCProd      |  Sim          |  Decimal   |  Até 12 dígitos, de 0 a 4 casas decimais
valor_pis_st                    |  vPIS         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais

### situacao_tributaria = 49, 50, 51, 52, 53, 54, 55, 56, 60, 61, 62, 63, 64, 65, 66, 67, 70, 71, 72, 73, 74, 75, 98, 99

49 = Outras Operações de Saída
50 = Operação com Direito a Crédito - Vinculada Exclusivamente a Receita Tributada no Mercado Interno
51 = Operação com Direito a Crédito – Vinculada Exclusivamente a Receita Não Tributada no Mercado Interno
52 = Operação com Direito a Crédito - Vinculada Exclusivamente a Receita de Exportação
53 = Operação com Direito a Crédito - Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno
54 = Operação com Direito a Crédito - Vinculada a Receitas Tributadas no Mercado Interno e de Exportação
55 = Operação com Direito a Crédito - Vinculada a Receitas Não-Tributadas no Mercado Interno e de Exportação
56 = Operação com Direito a Crédito - Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno, e de Exportação
60 = Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita Tributada no Mercado Interno
61 = Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita Não-Tributada no Mercado Interno
62 = Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita de Exportação
63 = Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno
64 = Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas no Mercado Interno e de Exportação
65 = Crédito Presumido - Operação de Aquisição Vinculada a Receitas Não-Tributadas no Mercado Interno e de Exportação
66 = Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno, e de Exportação
67 = Crédito Presumido - Outras Operações
70 = Operação de Aquisição sem Direito a Crédito
71 = Operação de Aquisição com Isenção
72 = Operação de Aquisição com Suspensão
73 = Operação de Aquisição a Alíquota Zero
74 = Operação de Aquisição sem Incidência da Contribuição
75 = Operação de Aquisição por Substituição Tributária
98 = Outras Operações de Entrada
99 = Outras Operações

### Cálculo por percentual

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
valor_base_calculo              |  vBC          |  Sim          |  Decimal   |   Até 13 dígitos, 2 casas decimais
aliquota_pis                    |  pPIS         |  Sim          |  Decimal   |   Até 3 dígitos, 4 casas decimais
valor_pis                       |  vPIS         |  Sim          |  Decimal   |   Até 13 dígitos, 2 casas decimais

### Cálculo por valor

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
aliquota_pis_reais              |  vAliqProd    |  Sim          |  Decimal   |  Até 11 dígitos, de 0 a 4 casas decimais
quantidade_vendida              |  qBCProd      |  Sim          |  Decimal   |  Até 12 dígitos, de 0 a 4 casas decimais
valor_pis                       |  vPIS         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais

### cofins (XML: COFINS)

Atributos comuns a todas as situações tributárias

### situacao_tributaria

 Campo XML  |  Obrigatório  |  Tipo      |  Formato e tamanho |  Observações
------------|---------------|------------|--------------------|------------
CST         |  Sim          |  Numérico  |  2 dígitos         |

### situacao_tributaria = 01 e 02

01 = Operação Tributável com Alíquota Básica
02 = Operação Tributável com Alíquota Diferenciada

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
valor_base_calculo              |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
aliquota_cofins                 |  pCOFINS      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais
valor_cofins                    |  vCOFINS      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais

### situacao_tributaria = 03

Operação Tributável com Alíquota por Unidade de Medida de Produto.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
aliquota_cofins_reais           |  vAliqProd    |  Sim          |  Decimal   |  Até 11 dígitos, de 0 a 4 casas decimais |
quantidade_vendida              |  qBCProd      |  Sim          |  Decimal   |  Até 12 dígitos, de 0 a 4 casas decimais |
valor_cofins                    |  vCOFINS      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais        |

### situacao_tributaria = 04, 06, 07, 08, 09

04 = Operação Tributável Monofásica - Revenda a Alíquota Zero
06 = Operação Tributável a Alíquota Zero
07 = Operação Isenta da Contribuição
08 = Operação sem Incidência da Contribuição
09 = Operação com Suspensão da Contribuição

Não há campos para informar alíquota.

### situacao_tributaria = 05

Operação Tributável por Substituição Tributária.

### Cálculo por percentual

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
valor_base_calculo_st           |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais        |
aliquota_st                     |  pCOFINS      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais         |
valor_cofins_st                 |  vCOFINS      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais        |


### Cálculo por valor

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
aliquota_cofins_reais_st        |  vAliqProd    |  Sim          |  Decimal   |  Até 11 dígitos, de 0 a 4 casas decimais
quantidade_vendida_st           |  qBCProd      |  Sim          |  Decimal   |  Até 12 dígitos, de 0 a 4 casas decimais
valor_cofins_st                 |  vCOFINS      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais

### situacao_tributaria = 49, 50, 51, 52, 53, 54, 55, 56, 60, 61, 62, 63, 64, 65, 66, 67, 70, 71, 72, 73, 74, 75, 98, 99

49 = Outras Operações de Saída
50 = Operação com Direito a Crédito - Vinculada Exclusivamente a Receita Tributada no Mercado Interno
51 = Operação com Direito a Crédito – Vinculada Exclusivamente a Receita Não Tributada no Mercado Interno
52 = Operação com Direito a Crédito - Vinculada Exclusivamente a Receita de Exportação
53 = Operação com Direito a Crédito - Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno
54 = Operação com Direito a Crédito - Vinculada a Receitas Tributadas no Mercado Interno e de Exportação
55 = Operação com Direito a Crédito - Vinculada a Receitas Não-Tributadas no Mercado Interno e de Exportação
56 = Operação com Direito a Crédito - Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno, e de Exportação
60 = Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita Tributada no Mercado Interno
61 = Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita Não-Tributada no Mercado Interno
62 = Crédito Presumido - Operação de Aquisição Vinculada Exclusivamente a Receita de Exportação
63 = Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno
64 = Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas no Mercado Interno e de Exportação
65 = Crédito Presumido - Operação de Aquisição Vinculada a Receitas Não-Tributadas no Mercado Interno e de Exportação
66 = Crédito Presumido - Operação de Aquisição Vinculada a Receitas Tributadas e Não-Tributadas no Mercado Interno, e de Exportação
67 = Crédito Presumido - Outras Operações
70 = Operação de Aquisição sem Direito a Crédito
71 = Operação de Aquisição com Isenção
72 = Operação de Aquisição com Suspensão
73 = Operação de Aquisição a Alíquota Zero
74 = Operação de Aquisição sem Incidência da Contribuição
75 = Operação de Aquisição por Substituição Tributária
98 = Outras Operações de Entrada
99 = Outras Operações

### Cálculo por percentual

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
valor_base_calculo              |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
aliquota_cofins                 |  pCOFINS      |  Sim          |  Decimal   |  Até 3 dígitos, 4 casas decimais
valor_cofins                    |  vCOFINS      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais

### Cálculo por valor

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
aliquota_cofins_reais           |  vAliqProd    |  Sim          |  Decimal   |  Até 11 dígitos, de 0 a 4 casas decimais
quantidade_vendida              |  qBCProd      |  Sim          |  Decimal   |  Até 12 dígitos, de 0 a 4 casas decimais
valor_cofins                    |  vCOFINS      |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais


### importacao (XML: II)

Contém informações sobre imposto de importação. Informar apenas quando o item for sujeito ao II.

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
base_calculo_importacao         |  vBC          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
valor_despesas_aduaneiras       |  vDespAdu     |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais | **Obrigatório quando usar engine de cálculo taxweb** |
valor_imposto_importacao        |  vII          |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais
valor_iof                       |  vIOF         |  Sim          |  Decimal   |  Até 13 dígitos, 2 casas decimais | **Obrigatório quando usar engine de cálculo taxweb** |

## declaracao_importacao (XML: DI)

Contém informações sobre as declarações de importação contidos na NF-e. No XML, o nó DI é subitem do nó prod e pode conter uma ou mais ocorrência. Seus atributos são:

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
documento_importacao            |  nDI          |  Sim          |  Numérico  |  De 1 a 12 dígitos
data_importacao                 |  dDI          |  Sim          |  Data      |  “AAAA-MM-DD”
local_desembaraco               |  xLocDesemb   |  Sim          |  Texto     |  1 a 60 caracteres
uf_desembaraco                  |  UFDesemb     |  Sim          |  Numérico  |  2 dígitos
data_desembaraco                |  dDesemb      |  Sim          |  Data      |  “AAAA-MM-DD”
via_transporte                  |  tpViaTransp  |  Sim          |  Numérico  |  2 dígitos | 1=Marítima;<br>2=Fluvial;<br>3=Lacustre;<br>4=Aérea;<br>5=Postal<br>6=Ferroviária;<br>7=Rodoviária;<br>8=Conduto / Rede Transmissão;<br>9=Meios Próprios;<br>10=Entrada / Saída ficta.<br>11=Courier;<br>12=Handcarry
valor_afrmm                     |  vAFRMM       |  Não          |  Decimal   |  Até 13 dígitos, 2 casas decimais
forma_importacao                |  tpIntermedio |  Sim          |  Numérico  |  1 dígito
cnpj                            |  CNPJ         |  Não          |  Numérico  |  14 dígitos
uf_adquirente                   |  UFTerceiro   |  Não          |  Decimal   |  2 dígitos
codigo_exportador               |  cExportador  |  Sim          |  Texto     |  1 a 60 caracteres

## medicamento (XML: med)

Contém informações sobre detalhamento de medicamentos e de matérias-primas farmacêuticas

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
codigo_anvisa | cProdANVISA | Sim | Texto | 6 a 13 caracteres
motivo_isencao | xMotivoIsencao | Não | Texto | 1 a 255 caracteres
preco_maximo | vPMC | Sim | Decimal | Até 13 dígitos, 2 casas decimais


### adicoes (XML: adi)

Contém informações sobre adições de uma declaração de importacao. No XML, o nó adi é subitem do nó DI e pode conter uma ou mais ocorrência. Seus atributos são:

 Campo                          |  XML          |  Obrigatório  |  Tipo      |  Formato e tamanho                       |  Observações
--------------------------------|---------------|---------------|------------|------------------------------------------|------------
numero_adicao                   |  nAdicao      |  Sim          |  Decimal   |  1 a 3 dígitos
numero_sequencial               |  nSeqAdic     |  Sim          |  Decimal   |  1 a 3 dígitos
codigo_fabricante               |  cFabricante  |  Sim          |  Decimal   |  1 a 60 caracteres
valor_desconto                  |  vDescDI      |  Não          |  Decimal   |  Até 13 dígitos, 2 casas decimais
numero_drawback                 |  nDraw        |  Não          |  Numérico  |  9 (AANNNNNND) ou 11 dígitos (AAAANNNNNND)


## transporte (XML: transp)

Contém informações sobre o frete e o transporte dos produtos ou serviços.

Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------------|-----------------------------------------------------------
codigo_modalidade           |  modFrete       |  Sim          |  Numérico               |  1 dígito                          |  Seleção entre:<br>0 = Por conta do emitente;<br>1 = Por conta do destinatário/remetente;<br>2 = Por conta de terceiros;<br>9 = Sem frete.<br>
valor_total_frete           |  vServ          |  Sim          |  Decimal                |  Até 13 dígitos, 2 casas decimais

### transportadora (XML: transporta)

Grupo de informações relacionadas à transportadora, caso haja frete.

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------------|-----------------------------------------------------------
    razao_social                |  xNome          |  Não          |  Texto                  |  Até 60 caracteres                 |  Nome (pessoa física) ou Razão Social (Pessoa Jurídica).
    cnpj_cpf                    |  CNPJ ou CPF    |  Não          |  Numérico               |  14 ou 11 dígitos                  |
    inscricao_estadual          |  IE             |  Não          |  Texto e/ou número      |  De 2 a 14 caracteres              |  Inscrição Estadual do transportador contribuinte do ICMS, sem caracteres de formatação (ponto, barra, hífen, etc.). Pode ser informado o texto “ISENTO” para transportador isento de inscrição no cadastro de contribuintes ICMS. Não informar a tag para não contribuinte do ICMS. A UF deve ser informada se informado uma IE.
    endereco_completo           |  xEnder         |  Não          |  Texto                  |  Até 60 caracteres                 |
    uf                          |  UF             |  Não          |  Texto                  |  2 caracteres                      |  Sigla da UF
    municipio                   |  xMun           |  Não          |  Texto                  |  Até 60 caracteres                 |

### retencao_icms (XML: retTransp)

Informações de retenção de ICMS de transporte.

    Campo                       |  Campo no XML   |  Obrigatório  |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------|-------------------------|------------------------------------|-----------------------------------------------------------
    cfop                        |  CFOP           |  Não          |  Numérico               |  4 dígitos                         |  CFOP de serviço de transporte.
    valor_base_calculo          |  vBCRet         |  Sim          |  Decimal                |  Até 13 dígitos, 2 casa decimais   |
    aliquota_retencao           |  pICMSRet       |  Sim          |  Decimal                |  Até 3 dígitos, de 2 a 4 casas decimais|
    valor_retido                |  vICMSRet       |  Sim          |  Decimal                |  Até 13 dígitos, 2 casas decimais  |
    uf                          |  -              |  Não          |  Texto                  |  2 caracteres                      |  Sigla da UF
    codigo_municipio            |  cMunFG         |  Sim          |  Numérico               |  7 dígitos                         |  Código do município de acordo com tabela do IBGE

### veiculo (XML: veicTransp)

Dados do veículo utilizado para transporte, caso haja frete. O campo pode ser omitido caso não haja frete.
placa

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    placa                       |  Sim            |  Texto e/ou número  |  7 caracteres           |                                    |  Informar em um dos seguintes formatos: XXX9999, XXX999, XX9999 ou XXXX999. Informar a placa em informações complementares quando a placa do veículo tiver lei de formação diversa.
    uf                          |  UF             |  Sim                |  Texto                  |  2 caracteres                      |  Sigla da UF
    rntc                        |  RNTC           |  Não                |  Texto                  |  Até 20 caracteres                 |  Registro Nacional de Transporte de Carga.

### volume_transportado (XML: vol)

Informações dos volumes transportados, se houver. Uma nota comporta até 5.000 volumes. Cada volume deve possuir os conjuntos de campos a seguir:

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    quantidade_volumes          |  qVol           |  Não                |  Numérico               |  Até 15 dígitos
    especie                     |  esp            |  Não                |  Texto                  |  Até 60 caracteres
    marca                       |  marca          |  Não                |  Texto                  |  Até 60 caracteres
    numeracao_volumes           |  nVol           |  Não                |  Texto                  |  Até 60 caracteres                  |  Numeração dos volumes transportados
    peso_liquido                |  peslL          |  Não                |  Decimal                |  Até 12 dígitos, 3 casas decimais   |  Peso líquido em quilogramas (kG).
    peso_bruto                  |  pesoB          |  Não                |  Decimal                |  Até 12 dígitos, 3 casas decimais   |  Peso bruto em quilogramas (kG).

### lacres (XML: lacres)

Cada volume transportador pode conter lacres, sendo até 5.000 por volume.

### Item da coleção (array) de lacres

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    Número do lacre             |  nLacre         |  Sim                |  Texto                  |  Até 60 caracteres                 |

## cobranca (XML: cobr)

Informações opcionais de cobrança. Seus atributos são:

### fatura (XML: fat)

Subgrupo de informações relacionadas aos dados da fatura. Possui somente uma ocorrência. Apresenta os seguintes atributos:

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |   Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    numero_fatura               |  nFat           |  Não                |  Texto e/ou número      |  Até 60 caracteres                 |  Número da fatura.
    valor_original              |  vOrig          |  Não                |  Decimal                |  13 dígitos, 2 casas decimais      |  Valor original (bruto) da fatura.
    valor_desconto              |  vDesc          |  Não                |  Decimal                |  13 dígitos, 2 casas decimais      |  Valor aplicado de desconto.
    valor_liquido               |  vLiq           |  Não                |  Decimal                |  13 dígitos, 2 casas decimais      |  Valor líquido da fatura.

### duplicata (XML: dup)

Subgrupo de informações relacionadas às duplicatas (parcelas da compra). Pode conter até 200 ocorrências e possui os seguintes atributos:

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |  Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    numero                      |  nDup           |  Não                |  Texto e/ou número      |  Até 60 caracteres                 |  Número da duplicata.
    data_vencimento             |  dVenc          |  Não                |  Data                   |  AAAA-MM-DD                        |  Data de vencimento da duplicata
    valor                       |  vDup           |  Sim                |  Decimal                |  13 dígitos, 2 casas decimais      |  Valor da duplicata.

## documentos_referenciados (XML: NFref)

Este campo é utilizado para referenciar uma Nota Fiscal, modelo 1 ou 1-A, ou NF-e emitida anteriormente e vinculada à NF-e que está sendo emitida. Cabe observar que esta informação será utilizada quando o regulamento do ICMS determinar indicação dos dados de outra nota fiscal referente àquela operação, como, por exemplo, nos casos de devolução de mercadoria, complementação de imposto etc.

Seus atributos são:

### tipo_documento

Campo interno do Emites para indicar o tipo do documento. Os valores podem ser: NF-e, NF ou Cupom.

### tipo_documento = “NFe”

Para notas fiscais eletrônicas relacionadas, é necessário somente a chave de acesso.

### chave_acesso

Referencia uma NF-e (modelo 55) emitida anteriormente, vinculada à NF-e atual, ou uma NFC-e (modelo 65).

 Campo                          |  Obrigatório  |  Tipo               |  Formato e tamanho                  |  Observações
--------------------------------|---------------|---------------------|-------------------------------------|------------
refNFe                          |  Sim          |  Numérico           |  44 dígitos

### tipo_documento = “NF”

Para notas fiscais físicas, os atributos são:

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |  Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    codigo_uf_emitente          |  cUF            |  Sim                |  Numérico               |  2 dígitos                         |  Código de acordo com tabela IBGE
    ano_mes_emissao             |  AAMM           |  Sim                |  Numérico               |  ano (AA) e mês (MM) de emissão da NF
    modelo_documento            |  mod            |  Sim                |  Numérico               |  2 dígitos                         |  01 = modelo 01
    numero_documento            |  nNF            |  Sim                |  Numérico               |  Até 9 dígitos                     |
    serie_documento             |  serie          |  Sim                |  Numérico               |  Até 3 dígitos                     |  Informar zero se não utilizada a série do documento fiscal.
    cnpj_emitente               |  CNPJ           |  Sim                |  Numérico               |  14 dígitos

### tipo_documento = “Cupom”

Além dos campos similares à nota fiscal física, o cupom fiscal tem os seguintes atributos:

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |  Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    numero_ecf                  |  nECF           |  Sim                |  Numérico               |  3 dígitos                         |  Número de ordem sequencial de ECF.
    numero_coo                  |  nCOO           |  Sim                |  Numérico               |  6 dígitos                         |  Número do Contador de Ordem de Operação (COO).

## retencao_tributos (XML: retTrib)

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |  Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    valor_retido_pis            |  vRetPIS        |  Não                |  Decimal                |  13 dígitos, 2 casas decimais      |
    valor_retido_cofins         |  vRetCOFINS     |  Não                |  Decimal                |  13 dígitos, 2 casas decimais      |
    valor_retido_csll           |  vRetCSLL       |  Não                |  Decimal                |  13 dígitos, 2 casas decimais      |
    valor_base_calculo_irrf     |  vBCIRRF        |  Não                |  Decimal                |  13 dígitos, 2 casas decimais      |
    valor_irrf                  |  vIRRF          |  Não                |  Decimal                |  13 dígitos, 2 casas decimais      |
    base_calculo_retencao_previdencia|vBCRetPrev  |  Não                |  Decimal                |  13 dígitos, 2 casas decimais      |
    valor_retencao_previdencia  |  vRetPrev       |  Não                |  Decimal                |  13 dígitos, 2 casas decimais      |

## exporta (XML: exporta)

Informações de comércio exterior.

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |  Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    uf_embarque                 |  UFSaidaPais    |  Sim                |  Texto                  |  2 caracteres                      |  Sigla da UF de Embarque ou de transposição de fronteira.
    local_embarque              |  xLocExporta    |  Sim                |  Texto                  |  Até 60 caracteres                 |  Descrição do Local de Embarque ou de transposição de fronteira.
    local_despacho              |  xLocDespacho   |  Sim                |  Texto                  |  Até 60 caracteres                 |  Descrição do local de despacho.

## forma_de_pagamento (XML: pag)

Grupo de Detalhamento da Forma de Pagamento. No XML, o nó detPag é subitem do nó pag e pode conter uma ou mais ocorrência. Seus atributos são:

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |  Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    tipo_de_pagamento           |  tPag           |  Sim                |  Texto                  |  2 caracteres                      |  01=Dinheiro;<br>02=Cheque;<br>03=Cartão de Crédito;<br>04=Cartão de Débito;<br>05=Crédito Loja;<br>10=Vale Alimentação;<br>11=Vale Refeição;<br>12=Vale Presente;<br>13=Vale Combustível;<br>14=Duplicata Mercantil;<br>15=Boleto Bancário;<br>90= Sem pagamento;<br>99=Outros.
    valor_do_pagamento          |  vPag           |  Sim                |  Decimal                | 13 dígitos, 2 casas decimais       |

## cartao (XML: card)

Grupo de Cartões. No XML, o nó card é subitem do nó detPag e pode conter apenas uma ocorrência. Seus atributos são:

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |  Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    tipo_de_integracao          |  tpIntegra      |  Sim                |  Numérico               | 1 dígito                           | 1=Pagamento integrado com o sistema de automação da empresa (Ex.: equipamento TEF, Comércio Eletrônico);<br>2= Pagamento não integrado com o sistema de automação da empresa (Ex.: equipamento POS);
    cnpj_credenciadora          |  CNPJ           |  Não                |  Numérico               | 14 dígitos                         |
    bandeira_operadora          |  tBand          |  Não                |  Numérico               | 2 caracteres                       | 01=Visa;<br>02=Mastercard;<br>03=American Express;<br>04=Sorocred;<br>05=Diners Club;<br>06=Elo;<br>07=Hipercard;<br>08=Aura;<br>09=Cabal;<br>99=Outros.
    numero_autorizacao_operacao |  cAut           |  Não                |  Numérico               | Até 20 caracteres                  |

## informacoes_adicionais (XML: infAdic)

Grupo de informações adicionais da NF-e. Seus atributos são:

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |  Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
    informacoes_contribuinte    |  infCpl         |  Não                |  Texto                  |  Até 2.000 caracteres              |  Informações complementares de interesse do Contribuinte.
    informacoes_fisco           |    infAdFisco   |  Não                |  Texto                  |  Até 2.000 caracteres              |  Informações adicionais de interesse do Fisco.

## responsavel_tecnico (XML: infRespTec)

<aside class="warning">
  Esses campos não devem ser enviados na requisição.
</aside>

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |  Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
 cnpj | CNPJ | - | - | - | CNPJ da pessoa jurídica responsável pelo sistema utilizado na emissão do documento fiscal eletrônico
 contato | xContato | - | - | - | Nome da pessoa a ser contatada na empresa desenvolvedora do sistema utilizado na emissão do documento fiscal eletrônico
 email | email | - | - | - | E-mail da pessoa a ser contatada na empresa desenvolvedora do sistema.
 fone | fone | - | - | - | O telefone da pessoa a ser contatada na empresa desenvolvedora do sistema.
 id_csrt | idCSRT | - | - | - | Identificador do CSRT utilizado para montar o hash do CSRT
 hash_csrt | hashCSRT | - | - | - | -

## resposta_emissao (XML: N/A)

Grupo informativo de resposta da emissao da NF-e.

<aside class="warning">
  Esses campos não devem ser enviados na requisição.
</aside>

    Campo                       |  Campo no XML   |  Obrigatório        |     Tipo                |    Formato e tamanho               |  Observações
--------------------------------|-----------------|---------------------|-------------------------|------------------------------------|-----------------------------------------------------------
 data_emissao | dhRecbto | - | - | - | Preenchido com a data e hora do processamento
 codigo_verificacao | digVal | - | - | - | Digest Value da NF-e processada
 numero_protocolo | nProt | - | - | - | Número do Protocolo da NF-e
 chave_acesso | chNFe | - | - | - | Chave de Acesso da NF-e
 erros | - | - | - | - | Lista erros da emissão, podendo ser uma mensagem de rejeição da SEFAZ ou uma lista de erros de validação de Schema




## NF-e cancellation

To cancel an NF-e, send a message with the identifier `CANCEL_NFE`. The same identifier will be returned in the response.

The JSON payload should follow the format:

```
{ "chave_acesso": "53180922769530000131651110000001281355486170", "motivo": "Desistencia do comprador" }
```

where:

- `chave_acesso` is the NFC-e's access key (obtained in the creation response);
- `reason` is the description of the reason for the cancellation (optional; if informed should be between 15 and 255 characters in size)

The cancellation response's JSON payload will be similar to the one returned during the
cancellation NFC-e operation.



## Tax calculation

To calculate the taxes for an NFC-e (Electronic Consumer Invoice), send a message with the identifier `CALCULATE`. The same identifier will be returned in response.

Use as reference the documents used to create an NFCE-e by the Emites Client.



## Useful links

- [JSON schema](https://json-schema.org/): Documentation of vocabulary used for JSON schemas;
- [JSON schema validator](https://www.jsonschemavalidator.net/): allows you to validate a JSON string against its schema;
