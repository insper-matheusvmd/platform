# Exchange Service

## Objetivo

O `exchange-service` consulta taxas de câmbio entre moedas e disponibiliza esses dados para a plataforma. Ele recebe a moeda de origem e a moeda de destino pela rota, consulta a AwesomeAPI e retorna os valores de compra, venda e data da cotação.

## Tecnologias

| Tecnologia | Uso |
|---|---|
| FastAPI | Aplicação principal e endpoints REST |
| Uvicorn | Servidor ASGI |
| HTTPX | Cliente HTTP assíncrono para a API externa |
| Docker | Empacotamento da aplicação |
| Kubernetes | Deploy no EKS |
| Jenkins | Pipeline de build, push e deploy |

## Endpoints

| Método | Rota | Descrição |
|---|---|---|
| `GET` | `/health-check` | Verifica se o serviço está respondendo. |
| `GET` | `/exchanges/{from_currency}/{to_currency}` | Consulta a cotação entre duas moedas. |

## Entrada da consulta

Exemplo de chamada:

```http
GET /exchanges/USD/BRL
id-account: uuid-da-conta
```

Parâmetros:

| Campo | Descrição |
|---|---|
| `from_currency` | Moeda de origem, por exemplo `USD`. |
| `to_currency` | Moeda de destino, por exemplo `BRL`. |
| `id-account` | Header com o identificador da conta autenticada, repassado pelo gateway. |

## Saída

```json
{
  "sell": 5.68,
  "buy": 5.67,
  "date": "2026-05-30 18:20:00",
  "id-account": "uuid-da-conta"
}
```

## Integração externa

O serviço consulta a AwesomeAPI usando o par de moedas informado na rota:

```python
AWESOME_API_URL = "https://economia.awesomeapi.com.br/json/last/{from_currency}-{to_currency}"
```

Antes da chamada, as moedas são normalizadas para letras maiúsculas. Se a API externa falhar, o serviço retorna erro `502`. Se o par de moedas não existir na resposta, retorna `404`.

## Arquivos principais

| Arquivo | Função |
|---|---|
| `api/exchange-service/main.py` | Aplicação FastAPI, endpoints e integração externa. |
| `api/exchange-service/requirements.txt` | Dependências Python. |
| `api/exchange-service/Dockerfile` | Imagem Docker do serviço. |
| `api/exchange-service/Jenkinsfile` | Pipeline de build, push da imagem e deploy no EKS. |
| `api/exchange-service/k8s/deployment.yaml` | Deployment Kubernetes do serviço. |
| `api/exchange-service/k8s/service.yaml` | Service interno `exchange-service`. |

## Deploy e CI/CD

O `exchange-service` possui imagem Docker, pipeline Jenkins e manifests Kubernetes. Dentro do cluster, o gateway acessa o serviço pelo endereço interno `http://exchange-service:8080`, expondo a rota `/exchanges/**` para a aplicação.

## Trecho de código principal

```python
@app.get("/exchanges/{from_currency}/{to_currency}")
async def get_exchange(
    from_currency: str,
    to_currency: str,
    id_account: str = Header(...),
):
    from_currency = from_currency.upper()
    to_currency = to_currency.upper()
    pair_key = f"{from_currency}{to_currency}"
    url = AWESOME_API_URL.format(from_currency=from_currency, to_currency=to_currency)

    async with httpx.AsyncClient() as client:
        response = await client.get(url)

    if response.status_code != 200:
        raise HTTPException(status_code=502, detail=f"Failed to fetch exchange rate for {from_currency}/{to_currency}")

    data = response.json()

    if pair_key not in data:
        raise HTTPException(status_code=404, detail=f"Currency pair {from_currency}/{to_currency} not found")

    pair = data[pair_key]

    return {
        "sell": float(pair["ask"]),
        "buy": float(pair["bid"]),
        "date": pair["create_date"],
        "id-account": id_account,
    }
```
