<img alt="GoStack" src="https://scontent.fgyn21-1.fna.fbcdn.net/v/t1.6435-9/143499329_347249869697972_1658415451933314067_n.png?stp=dst-png_s960x960&_nc_cat=104&ccb=1-7&_nc_sid=e3f864&_nc_ohc=MyP9ttzI2KYAX9n0aYF&_nc_ht=scontent.fgyn21-1.fna&oh=00_AT8fCfMSeYsVG79fj14Hi7oO3ckbahI76428R9z6VV1Fxg&oe=62E3D64A" />

# Introdução

Este documento visa apresentar os recursos disponibilizados pela [Combux](https://www.combux.com.br/) via `API` para integração com parceiros e interessados.

## Autorização

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

A seguir estão descritas as possíveis respostas à requisição acima, sempre no formato JSON:

### Se der algo errado:
```javascript
{
  "message" : string,
  "code" : number
}
```
O atributo `message` trará uma mensagem de erro correspondente o e `code` trará o código HTTP do erro gerado por esta requisição.

### Se der tudo certo:
```javascript
{
  "authorization_code" : string
}
```
O atributo recebido `authorization_code` te garante passar para a próxima etapa de autenticação que é a obtenção `access_token(token de acesso)` e do `refresh_token(para atualizar seu token de acesso gerado acima quando o mesmo expirar)`.


#### 2. Gerando `access_token` e `refresh_token`:
O `header` desta requisição deve conter as seguintes informações:
```http
Content-Type: application/json;Authorization: Basic client_id:client_secret
```
**Note**: Esse `client_id:client_secret` deve ser uma string convertida em Base64.


O `endpoint` para obtenção do `access_token` e `refresh_token` é: 
```http
POST /oauth/access-token
```

O corpo desta requisição deve conter: 

```javascript
{
  "authorization_code" : string,
  "token_type": string
}
```
| Parameter | Type | Required | Description
| :--- | :--- | :--- | :---
| `authorization_code` | `string` | **TRUE**| Código gerado no passo anterior.
| `token_type` | `string` | **TRUE**| Tipo do token. Deve ser `access_token`.

A seguir estão descritas as possíveis respostas à requisição acima, sempre no formato JSON:

### Se der algo errado:
```javascript
{
  "message" : string,
  "code" : number
}
```
O atributo `message` trará uma mensagem de erro correspondente o e `code` trará o código HTTP do erro gerado por esta requisição.

### Se der tudo certo:
```javascript
{
  "access_token": string,
  "refresh_token": string,
  "expires_in": nubmer
}
```
O atributo recebido `access_token` te garante acessar as informações a partir daqui, o `refresh_token` é o token utilizado para atualizar seu `access_token` quando o mesmo expirar e `expires_in` representa o tempo de expiração do seu token de acesso (sempre em segundos).

#### 3. Atualizando `access_token` através do `refresh_token`:
O `header` desta requisição deve conter as seguintes informações:
```http
Content-Type: application/json;Authorization: Basic client_id:client_secret
```
**Note**: Esse `client_id:client_secret` deve ser uma string convertida em Base64.


O `endpoint` para obtenção de um novo `access_token` é: 
```http
POST /oauth/access-token
```

O corpo desta requisição deve conter: 

```javascript
{
  "refresh_token" : string
}
```
| Parameter | Type | Required | Description
| :--- | :--- | :--- | :---
| `refresh_token` | `string` | **TRUE**| Token de atualização gerado junto do token de acesso.

A seguir estão descritas as possíveis respostas à requisição acima, sempre no formato JSON:

### Se der algo errado:
```javascript
{
  "message" : string,
  "code" : number
}
```
O atributo `message` trará uma mensagem de erro correspondente o e `code` trará o código HTTP do erro gerado por esta requisição.

### Se der tudo certo:
```javascript
{
  "access_token": string,
  "refresh_token": string,
  "expires_in": number
}
```
O atributo recebido `access_token` é o seu novo token de acesso, o `refresh_token` é o token utilizado para atualizar novamente seu `access_token` quando o mesmo expirar e `expires_in` representa o tempo de expiração do seu novo token de acesso (sempre em segundos).


## Status Codes

Gophish returns the following status codes in its API:

| Status Code | Description |
| :--- | :--- |
| 200 | `OK` |
| 201 | `CREATED` |
| 400 | `BAD REQUEST` |
| 404 | `NOT FOUND` |
| 500 | `INTERNAL SERVER ERROR` |
