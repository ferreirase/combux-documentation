<img alt="GoStack" src="https://scontent.fgyn21-1.fna.fbcdn.net/v/t1.6435-9/143499329_347249869697972_1658415451933314067_n.png?stp=dst-png_s960x960&_nc_cat=104&ccb=1-7&_nc_sid=e3f864&_nc_ohc=MyP9ttzI2KYAX9n0aYF&_nc_ht=scontent.fgyn21-1.fna&oh=00_AT8fCfMSeYsVG79fj14Hi7oO3ckbahI76428R9z6VV1Fxg&oe=62E3D64A" />

# :beginner: Introdução

Este documento visa apresentar os recursos disponibilizados pela [Combux](https://www.combux.com.br/) via `API` para integração com parceiros.

## :key: Autorização

É preciso estar devidamente `autenticado` e `autorizado` para ter acesso aos recursos disponibilizados em nossa `API`. 

Utilizamos o padrão [JWT(Json Web Token)](https://jwt.io/) na geração dos `tokens` de acesso juntamente com uma implementação de [OAuth2](https://oauth.net/2/). 

Nosso fluxo de `autenticação` e `autorização` está descrito abaixo. 

#### 1. Gerando o `authorization_code`:
O `header` desta requisição deve conter a seguinte informação:
```http
Content-Type: application/json
```
O `endpoint` para obtenção do `authorization_code` é: 
```http
POST /oauth/auth-code
```

O corpo desta requisição deve conter: 

```javascript
{
  "client_id" : string,
  "client_secret": string
}
```
| Parameter | Type | Required | Description
| :--- | :--- | :--- | :---
| `client_id` | `string` | **TRUE**| Disponibilizado em contrato.
| `client_secret` | `string` | **TRUE**| Disponibilizado em contrato.

A seguir estão descritas as possíveis respostas à requisição acima, sempre no formato `JSON`:

### :x: Se der algo errado:
```javascript
{
  "message" : string,
  "code" : number
}
```
O atributo `message` trará uma mensagem de erro correspondente o e `code` trará o código HTTP do erro gerado por esta requisição.

### :heavy_check_mark: Se der tudo certo:
```javascript
{
  "authorization_code" : string
}
```
O atributo recebido `authorization_code` te garante passar para a próxima etapa de autenticação que é a obtenção `access_token(token de acesso)` e do `refresh_token(para atualizar seu token de acesso gerado acima quando o mesmo expirar)`.


#### 2. Gerando `access_token` e `refresh_token`:
O `header` desta requisição deve conter as seguintes informações:
```http
Content-Type: application/json;Authorization: Bearer base64_string
```
:heavy_exclamation_mark: **Nota**: Esse `base64_string` deve ser uma string convertida em Base64 a partir do dado `client_id:client_secret`.


O `endpoint` para obtenção do `access_token` e `refresh_token` é: 
```http
POST /oauth/access-token
```

O corpo desta requisição deve conter: 

```javascript
{
  "code" : string,
  "grant_type": string
}
```
| Parameter | Type | Required | Description
| :--- | :--- | :--- | :---
| `authorization_code` | `string` | **TRUE**| Código gerado no passo anterior.
| `grant_type` | `string` | **TRUE**| Tipo da solicitação. Deve ser `authorization_code`.

A seguir estão descritas as possíveis respostas à requisição acima, sempre no formato JSON:

### :x: Se der algo errado:
```javascript
{
  "message" : string,
  "code" : number
}
```
O atributo `message` trará uma mensagem de erro correspondente o e `code` trará o código HTTP do erro gerado por esta requisição.

### :heavy_check_mark: Se der tudo certo:
```javascript
{
  "access_token": string,
  "refresh_token": string,
  "expires_in": number
}
```
O atributo recebido `access_token` te garante acessar as informações a partir daqui, o `refresh_token` é o token utilizado para atualizar seu `access_token` quando o mesmo expirar e `expires_in` representa o tempo de expiração do seu token de acesso (sempre em segundos).

#### 3. Atualizando `access_token` através do `refresh_token`:
O `header` desta requisição deve conter as seguintes informações:
```http
Content-Type: application/json;Authorization: Bearer base64_string
```
:heavy_exclamation_mark: **Nota**: Esse `base64_string` deve ser uma string convertida em Base64 a partir do dado `client_id:client_secret`.


O `endpoint` para obtenção de um novo `access_token` é: 
```http
POST /oauth/access-token
```

O corpo desta requisição deve conter: 

```javascript
{
  "grant_type": string,
  "refresh_token" : string
}
```
| Parameter | Type | Required | Description
| :--- | :--- | :--- | :---
| `grant_type` | `string` | **TRUE**| Tipo da solicitação. Deve ser `refresh_token`.
| `refresh_token` | `string` | **TRUE**| Token de atualização gerado junto do token de acesso.

A seguir estão descritas as possíveis respostas à requisição acima, sempre no formato JSON:

### :x: Se der algo errado:
```javascript
{
  "message" : string,
  "code" : number
}
```
O atributo `message` trará uma mensagem de erro correspondente o e `code` trará o código HTTP do erro gerado por esta requisição.

### :heavy_check_mark: Se der tudo certo:
```javascript
{
  "access_token": string,
  "refresh_token": string,
  "expires_in": number
}
```
O atributo recebido `access_token` é o seu novo token de acesso, o `refresh_token` é o token utilizado para atualizar novamente seu `access_token` quando o mesmo expirar e `expires_in` representa o tempo de expiração do seu novo token de acesso (sempre em segundos).

## :open_file_folder: Recursos

Depois de autenticado e autorizado, os seguintes recursos, nas suas respectivas rotas, estarão disponíveis para serem acessados:

1. Atualização de preços por foto


O `header` desta requisição deve conter as seguintes informações:
```http
Content-Type: multipart/form-data;Authorization: Bearer access_token
```

```http
POST /prices/photo
```

Os seguintes dados devem ser enviados juntos no form-data: 

```javascript
{
  "gas_station" : {
    "cod_unity": number,
    "cnpj": string,
    "terms_of_use": boolean,
  },
  "coordinates?": {
    "latitude": string,
    "longitude": string
  },
  "photo": file
}
```

| Parameter | Type | Required | Description
| :--- | :--- | :--- | :---
| `gas_station.cod_unity` | `string` | **TRUE**| Código que identifica um posto como único.
| `gas_station.cnpj` | `string` | **TRUE**| `CNPJ` do posto(com máscara).
| `coordinates.latitude` | `string` | **FALSE**| Latitude na localização do posto.
| `coordinates.longitude` | `string` | **FALSE**| Longitude na localização do posto.

A seguir estão descritas as possíveis respostas à requisição acima, sempre no formato JSON:

### :x: Se der algo errado:
```javascript
{
  "message" : string,
  "code" : number
}
```
O atributo `message` trará uma mensagem de erro correspondente o e `code` trará o código HTTP do erro gerado por esta requisição.

### :heavy_check_mark: Se der tudo certo:
```javascript
{
  "gas_station" : {
      "cod_unity": number,
      "cnpj": string,
      "terms_of_use": boolean,
      "coordinates?": {
        "latitude": string,
        "longitude": string
       },
    },
    "photo_url": string,
    "created_at": string,
    "status" 
    "prices: [
      {
        "type": number/int,
        "price": number/float
      },
      ...
    ]
}
```

O objeto retornado `posto` contém as informações do posto,  o `created_at` é a data de criação da foto  no formato ISO e o `prices` é um array de preços extraídos da foto enviada.

A propriedade "type" dos objetos de "prices" será um number de acordo com a tabela de [Tipos de combustíveis](#tipos-de-combustíveis).

2. Atualização de preços manualmente


O `header` desta requisição deve conter as seguintes informações:
```http
Content-Type: application/json;Authorization: Bearer access_token
```

```http
POST /prices/manual
```

O corpo desta requisição deve conter: 

```javascript
{
  "gas_station" : {
    "cod_unity": number,
    "cnpj": string,
    "terms_of_use": boolean,
  },
  "coordinates?": {
    "latitude": string,
    "longitude": string
  },
  "prices": [
    {
      "type": number/int,
      "price": number/float
    },
    ...
  ]
}
```

| Parameter | Type | Required | Description
| :--- | :--- | :--- | :---
| `gas_station.cod_unity` | `string` | **TRUE**| Código que identifica um posto como único.
| `gas_station.cnpj` | `string` | **TRUE**| CNPJ do posto.
| `coordinates.latitude` | `string` | **FALSE**| Latitude na localização do posto.
| `coordinates.longitude` | `string` | **FALSE**| Longitude na localização do posto.
| `prices.type` | `number/int` | **TRUE**| Código do tipo de combustível.
| `prices.price` | `number/float` | **TRUE**| Preço do combustível.

A seguir estão descritas as possíveis respostas à requisição acima, sempre no formato JSON:

### :x: Se der algo errado:
```javascript
{
  "message" : string,
  "code" : number
}
```
O atributo `message` trará uma mensagem de erro correspondente o e `code` trará o código HTTP do erro gerado por esta requisição.

### :heavy_check_mark: Se der tudo certo:
```javascript
{
  "success": boolean
}
```

## Tipos de combustíveis

| Fuel code | Description |  Description |
| :--- | :--- | :--- |
| 1 | `GASOLINA_COMUM` | Gasolina comum
| 2 | `GASOLINA_ADITIVADA` | Gasolina aditivada 
| 3 | `GASOLINA_PREMIUM` | Gasolina premium
| 5 | `ETANOL` | Etanol comum
| 6 | `ETANOL_ADITIVADO` | Etanol aditivado
| 9 | `GNV` | Gás natural veicular
| 10 | `DIESEL_S10` | Diesel S-10
| 11 | `DIESEL_S10_ADITIVADO` | Diesel S-10 aditivado
| 12 | `DIESEL_S500` | Diesel S-500
| 13 | `DIESEL_S500_ADITIVADO` | Diesel S-500 aditivado


## Status Codes

| Status Code | Description |
| :--- | :--- |
| 🟢 200 | `OK` | Requisição realizada com sucesso
|  🟢 201 | `CREATED` | Recurso criado com sucesso
|:red_circle: 400 | `BAD REQUEST` | Erro na solicitação
| :red_circle: 401 | `UNAUTHORIZED` | Recurso não autorizado
| :red_circle: 404 | `NOT FOUND` | Recurso não encontrado
| :red_circle: 500 | `INTERNAL SERVER ERROR` | Erro interno da API
