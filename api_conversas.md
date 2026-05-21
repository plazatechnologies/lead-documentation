# API de Criação de Conversa

Endpoint para criar conversas e iniciar atendimento automatizado via WhatsApp.

## URL Base

```
https://app.useplaza.com.br
```

## Endpoint

```
POST /conversations/sdr
```

## Autenticação

Inclua seu token de API como parâmetro `token` na URL ou no corpo da requisição.

## Parâmetros da Requisição

| Parâmetro          | Tipo   | Descrição                                                                       |
| ------------------ | ------ | ------------------------------------------------------------------------------- |
| `phone`            | string | Telefone do cliente (formato E.164 recomendado, ex: `+5511999999999`). Obrigatório, mínimo 10 dígitos. |
| `listing_id`       | string | Código do imóvel de interesse do cliente                                        |
| `name`             | string | Nome do cliente                                                                 |
| `email`            | string | E-mail do cliente                                                               |
| `origin`           | string | Origem do lead (ex: `"facebook"`, `"website"`, `"landing_page"`)                |
| `campaign`         | string | Identificador da campanha para rastreamento                                     |
| `transaction_type` | string | `"sale"` (venda) ou `"rent"` (aluguel)                                          |

## Exemplos

### cURL

```bash
curl -X POST "https://app.useplaza.com.br/conversations/sdr?token=SEU_TOKEN_API" \
  -H "Content-Type: application/json" \
  -d '{
    "phone": "+5511999999999",
    "listing_id": "ABC123",
    "name": "João Silva",
    "email": "joao@example.com",
    "origin": "facebook",
    "transaction_type": "rent"
  }'
```

### Form Data

```bash
curl -X POST "https://app.useplaza.com.br/conversations/sdr" \
  -d "token=SEU_TOKEN_API" \
  -d "phone=+5511999999999" \
  -d "listing_id=ABC123" \
  -d "name=João Silva" \
  -d "origin=website" \
  -d "transaction_type=sale"
```

### JavaScript

```javascript
const response = await fetch('https://app.useplaza.com.br/conversations/sdr?token=SEU_TOKEN_API', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    phone: '+5511999999999',
    listing_id: 'ABC123',
    name: 'João Silva',
    email: 'joao@example.com',
    origin: 'website',
    transaction_type: 'rent'
  })
});

const data = await response.json();
```

### Python

```python
import requests

response = requests.post(
    "https://app.useplaza.com.br/conversations/sdr",
    params={"token": "SEU_TOKEN_API"},
    json={
        "phone": "+5511999999999",
        "listing_id": "ABC123",
        "name": "João Silva",
        "email": "joao@example.com",
        "origin": "google_ads",
        "transaction_type": "sale"
    }
)
print(response.json())
```

## Respostas

### Sucesso

```json
HTTP 200 OK

{
  "message": "Conversation started"
}
```

### Imóvel Não Encontrado

```json
HTTP 404 Not Found

{
  "error": "Listing not found"
}
```

### Parâmetros Inválidos

```json
HTTP 422 Unprocessable Entity

{
  "error": "Invalid parameters",
  "details": {
    "phone": "is required (e.g. '+5511999999999')",
    "email": "invalid format",
    "transaction_type": "invalid value 'foo', accepted: sale, rent"
  }
}
```

Retornado quando `phone` está ausente ou tem menos de 10 dígitos, quando `email` não está em formato válido, ou quando `transaction_type` tem valor não aceito. O objeto `details` inclui apenas os campos com erro.

### Não Autorizado

```
HTTP 401 Unauthorized
```

## Observações

- Números de telefone são automaticamente normalizados para o formato E.164
- O `listing_id` deve corresponder a um código de imóvel existente no seu inventário
- As conversas são criadas de forma assíncrona - uma resposta `200` confirma que a requisição foi aceita
