# Lead Creation Webhook Documentation

## Overview

Use this endpoint to create a new lead for a specific real estate. You must provide the required parameters and a valid authorization token.

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

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| clientListingId | string | No | Internal listing ID. |
| email | string | No | Lead’s email address. |
| name | string | Yes | Lead’s full name. |
| ddd | string | Yes | The area code of the lead's phone number. |
| phone | string | Yes | The lead’s phone number (excluding area code). |
| transactionType | string | No | Type of transaction: `rent` or `buy`. |
| origin | string | No | Source of the lead. Possible values: `WhatsApp`, `GrupoZap`, `ImovelWeb`, `ChavesNaMaoLais`, `VivaReal`. |
| leadSummary | string | No | A short summary highlighting the lead’s key details and interests. |
| leadPreferences | string | No | Specific property preferences indicated by the lead. |
| listingLink | string | No | URL pointing to the associated listing. |
| cpf/cnpj | string | No | The lead’s CPF or CNPJ (Brazilian ID numbers). |
| intendedMoveDate | string | No | The date the lead intends to move. |
| conversation | string | No | A URL linking to a conversation transcript with the lead. |
| visitDate | string | No | The date the lead plans to visit the property. |

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
  "transactionType": "buy",
  "origin": "WhatsApp",
  "leadSummary": "Interested in 2-bedroom apartments.",
  "leadPreferences": "Wants a balcony and near public transport.",
  "listingLink": "https://listing.example.com/property/ABC123",
  "cpf/cnpj": "123.456.789-00",
  "intendedMoveDate": "2025-01-15",
  "conversation": "https://useplaza.comv.br/conversations/123",
  "visitDate": "2024-12-20-12:10:30 -03:00"
}
```

## Responses

### 201 Created

When a lead is successfully created, the response includes detailed information about the new lead. This helps the caller confirm the correct processing of the data and provides a reference for future interactions.

**PS: Please always return the lead_id created on your system.**

**Example Response Body for 201 Created:**

```json
{
  "status": "success",
  "message": "Lead created successfully.",
  "data": {
	  "clientListingId": "ABC123",
	  "email": "lead@example.com",
	  "name": "John Doe",
	  "ddd": "11",
	  "phone": "999999999",
	  "transactionType": "buy",
	  "origin": "WhatsApp",
	  "leadSummary": "Interested in 2-bedroom apartments.",
	  "leadPreferences": "Wants a balcony and near public transport.",
	  "listingLink": "https://listing.example.com/property/ABC123",
	  "cpf/cnpj": "123.456.789-00",
	  "intendedMoveDate": "2025-01-15",
	  "conversation": "https://useplaza.comv.br/conversations/123",
	  "visitDate": "2024-12-20-12:10:30 -03:00"
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
