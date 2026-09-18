# CRM Integration Documentation

## Overview

This documentation describes the endpoints your CRM must provide to integrate with Plaza:

1. **Lead Creation Webhook** — to **receive our leads**. Our leads will be sent to the webhook via the payload described in the **Request Body Parameters** below.
2. **Realtor Resolver** — to tell us which realtor is already assisting a client, so the conversation is handed to that realtor. See [Realtor Resolver Endpoint](#realtor-resolver-endpoint).

You must provide us the required {{realEstateID}} and a valid authorization token to successfully integrate.

# Lead Creation Webhook

## Endpoint

**Method:** `POST`

**Production URL:** `https://plaza.services/api/public/v1/leads/webhook/{{realEstateID}}`

> Replace {{realEstateID}} with the unique identifier of the real estate.
> 

## Headers

| Header | Value | Description |
| --- | --- | --- |
| Content-Type | `application/json` | Indicates the request body is in JSON format. |
| Authorization | `Bearer <YOUR_TOKEN>` | Provide a valid bearer token or agreed-upon secret. |

## URL Parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| realEstateID | string | Yes | The unique ID of the real estate. This must be included in the request URL. |

## Request Body Parameters

### NOTE: **ALL** fields must accept all strings. 
### E.G. The field *cpf/cnpj* can be "not mentioned" and the field *visitDate* can be "as soon as possible". Optional fields may also be sent as `null`.
### Additional fields may be included in the payload; please ignore any field you don't recognize.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| clientListingId | string | No | Internal listing ID. |
| email | string | No | Lead’s email address. |
| name | string | Yes | Lead’s full name. |
| ddd | string | Yes | The area code of the lead's phone number. |
| phone | string | Yes | The lead’s phone number (excluding area code). |
| transactionType | string | No | Type of transaction: `rent` or `sale`. |
| origin | string | No | Source of the lead, as free text (e.g. `WhatsApp`, `Facebook`, `VivaReal`). |
| leadSummary | string | No | A short summary highlighting the lead’s key details and interests. |
| leadPreferences | string | No | Specific property preferences indicated by the lead. |
| listingLink | string | No | URL pointing to the associated listing. |
| cpf/cnpj | string | No | The lead’s CPF or CNPJ (Brazilian ID numbers). |
| intendedMoveDate | string | No | The date the lead intends to move. |
| conversation | string | No | A URL linking to a conversation transcript with the lead. |
| visitDate | string | No | The date the lead plans to visit the property. |
| message | string | No | A message containing all lead info. |
| realtorId | string | No | ID, in your system, of the realtor (broker) responsible for the lead. `null` when the realtor should be set automatically by your system's internal rules (e.g. your own distribution/rotation). |

## Example Request

```bash
POST <https://plaza.services/api/public/v1/leads/webhook/12345>
Authorization: Bearer YOUR_ACCESS_TOKEN
Content-Type: application/json

{
  "clientListingId": "ABC123",
  "email": "lead@example.com",
  "name": "John Doe",
  "ddd": "11",
  "phone": "999999999",
  "transactionType": "sale",
  "origin": "WhatsApp",
  "leadSummary": "Interessado em apartamentos de 2 quartos.",
  "leadPreferences": "Sala ampla com varanda, em um bairro tranquilo.",
  "listingLink": "https://listing.example.com/property/ABC123",
  "cpf/cnpj": "123.456.789-00",
  "intendedMoveDate": "2025-01-15",
  "conversation": "https://app.useplaza.com.br/public-conversation/abc123",
  "visitDate": "2024-12-20-12:10:30 -03:00",
  "realtorId": "4521",
  "message": "💼 Quem é John Doe? \nNão informado\n\n🏢 Quais são suas necessidades? \nEstá em busca de um apartamento de 2 quartos, com sala ampla e varanda, em um bairro tranquilo.\n\n📅 Data da visita: 26/02/2025\n\n*CPF/CNPJ*: 123.456.789-00\n\n🚛 Quando planeja se mudar: 01/04/2025\n\nOrigem: Facebook\n\n🏭 Imóvel : https://exemplo.com.br/imovel/1234\n\nCódigo do Imóvel: 1234\n\nPrimeiro Imóvel de Interesse: 5678\n\n💬 Ver Conversa: [Ver conversa](https://exemplo.com.br/conversa/9876)\n\nStatus do Lead: Aguardando aprovação"
}
```

## Responses

### 201 Created

When a lead is successfully created, the response includes detailed information about the new lead. This helps the caller confirm the correct processing of the data and provides a reference for future interactions.

**PS: Please always return the `lead_id` created on your system, at the top level of the response body.**

**Example Response Body for 201 Created:**

```json
{
  "status": "success",
  "message": "Lead created successfully.",
  "lead_id": "98765",
  "data": {
	  "clientListingId": "ABC123",
	  "email": "lead@example.com",
	  "name": "John Doe",
	  "ddd": "11",
	  "phone": "999999999",
	  "transactionType": "sale",
	  "origin": "WhatsApp",
	  "leadSummary": "Interessado em apartamentos de 2 quartos.",
	  "leadPreferences": "Sala ampla com varanda, em um bairro tranquilo.",
	  "listingLink": "https://listing.example.com/property/ABC123",
	  "cpf/cnpj": "123.456.789-00",
	  "intendedMoveDate": "2025-01-15",
	  "conversation": "https://app.useplaza.com.br/public-conversation/abc123",
	  "visitDate": "2024-12-20-12:10:30 -03:00",
	  "realtorId": "4521"
	}
}
```

### **400 Bad Request**

This response is returned when the request fails due to incorrect input or missing required fields. It specifies the exact errors, enabling the caller to correct them and retry the request.

**Example Response Body for 400 Bad Request:**

```json
{
  "status": "error",
  "message": "Request validation failed.",
  "errors": [
    {
      "field": "phone",
      "message": "Phone number is required."
    },
    {
      "field": "name",
      "message": "Name is required."
    }
  ]
}
```

# Realtor Resolver Endpoint

When a client contacts us, Plaza calls this endpoint **before** assigning a realtor. If the client already has a realtor assisting them in your CRM, we hand the conversation to that realtor, so returning clients keep the same realtor. If no realtor is returned, Plaza assigns one following the real estate's own distribution rules.

This endpoint is optional. It is only needed if you want the realtor assignment to follow your CRM.

## Endpoint

**Method:** `GET`

**Production URL:** `https://plaza.services/api/public/v1/leads/realtor/{{realEstateID}}`

> Replace {{realEstateID}} with the unique identifier of the real estate.

## Headers

| Header | Value | Description |
| --- | --- | --- |
| Authorization | `Bearer <YOUR_TOKEN>` | Same token used for the Lead Creation Webhook. |

## Query Parameters

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| phone | string | Yes | The client's phone number, digits only, including country and area code (e.g. `5511999999999`). |
| transactionType | string | No | Type of transaction: `rent` or `sale`. Use it if your CRM keeps a different realtor per transaction type. |

## Example Request

```bash
GET <https://plaza.services/api/public/v1/leads/realtor/12345?phone=5511999999999&transactionType=rent>
Authorization: Bearer YOUR_ACCESS_TOKEN
```

## Responses

### 200 OK — realtor found

```json
{
  "realtorId": "4521",
  "realtorEmail": "realtor@example.com"
}
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| realtorId | string | Yes | ID, in your system, of the realtor assisting the client. The same ID used in the `realtorId` field of the Lead Creation Webhook. |
| realtorEmail | string | No | The realtor's email. Used to identify the realtor when the ID is not recognized. |

### 200 OK — no realtor

```json
{
  "realtorId": null
}
```

Return this when the client is unknown, has no open service, or the realtor is no longer active. Plaza will then assign a realtor following the real estate's distribution rules.

### Rules

- Only return a realtor who is **active** and **currently assisting** this client.
- Respond within **5 seconds**. If the endpoint times out or returns an error, Plaza assigns a realtor using the real estate's distribution rules.
