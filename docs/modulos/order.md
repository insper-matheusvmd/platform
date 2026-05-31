# Order Service

## Objetivo

O `order-service` registra pedidos feitos por uma conta autenticada. Para criar um pedido, ele recebe os itens, consulta o `product-service`, valida se cada produto existe, verifica estoque disponível, define o status inicial e calcula os totais.

## Tecnologias

| Tecnologia | Uso |
|---|---|
| Spring Boot | Aplicação principal |
| Spring MVC | Endpoints REST |
| Spring Data JPA | Persistência |
| PostgreSQL | Banco de dados |
| Flyway | Migrações |
| OpenFeign | Comunicação HTTP com produtos |
| Bean Validation | Validação de entrada |
| Actuator + Prometheus | Métricas |

## Endpoints

| Método | Rota | Descrição |
|---|---|---|
| `POST` | `/orders` | Cria um pedido para a conta do header `id-account`. |
| `GET` | `/orders` | Lista pedidos da conta. |
| `GET` | `/orders/{id}` | Busca detalhes de um pedido da conta. |
| `GET` | `/orders/actuator/prometheus` | Exposição de métricas. |

## Entrada de criação

```json
{
  "items": [
    {
      "idProduct": "uuid-do-produto",
      "quantity": 2
    }
  ]
}
```

Headers necessários:

| Header | Descrição |
|---|---|
| `id-account` | Identificador da conta dona do pedido. |

Regras de validação:

- `items` não pode ser vazio.
- `idProduct` é obrigatório.
- `quantity` deve ser maior ou igual a `1`.

## Saída de criação

```json
{
  "id": "uuid-do-pedido",
  "createdAt": "2026-05-20T10:30:00",
  "status": "CREATED",
  "items": [
    {
      "id": "uuid-do-item",
      "product": {
        "id": "uuid-do-produto"
      },
      "quantity": 2,
      "totalUsd": 25.80
    }
  ],
  "totalUsd": 25.80
}
```

## Integração com Product Service

O `order-service` usa OpenFeign para chamar `GET /products/{id}` no `product-service`.

```java
@FeignClient(name = "product", url = "${clients.product.url}")
public interface ProductClient {
    @GetMapping("/products/{id}")
    ProductSnapshotOut findById(@PathVariable String id);
}
```

Caso o produto não exista, a criação do pedido retorna erro de negócio:

```java
catch (FeignException.NotFound ex) {
    throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "Product not found: " + idProduct);
}
```

Caso a quantidade pedida seja maior do que o estoque retornado pelo `product-service`, a criação também retorna erro de negócio.

## Persistência

O serviço usa o schema `orders` e duas tabelas principais:

- `orders.orders`: cabeçalho do pedido, conta, data e total.
- `orders.order_items`: itens do pedido, produto, quantidade, preço unitário e total.

```sql
CREATE TABLE IF NOT EXISTS orders.orders (
    id VARCHAR(36) PRIMARY KEY,
    account_id VARCHAR(36) NOT NULL,
    created_at TIMESTAMP NOT NULL,
    status VARCHAR(32) NOT NULL,
    total_usd NUMERIC(12, 2) NOT NULL
);

CREATE TABLE IF NOT EXISTS orders.order_items (
    id VARCHAR(36) PRIMARY KEY,
    order_id VARCHAR(36) NOT NULL REFERENCES orders.orders(id) ON DELETE CASCADE,
    product_id VARCHAR(36) NOT NULL,
    quantity INTEGER NOT NULL,
    unit_price_usd NUMERIC(12, 2) NOT NULL,
    total_usd NUMERIC(12, 2) NOT NULL
);
```

A evolução `V2__add_status_to_orders.sql` adiciona o campo `status` em bancos já existentes.

## Arquivos principais

| Arquivo | Função |
|---|---|
| `api/order/src/main/java/store/order/OrderApplication.java` | Inicialização da aplicação. |
| `api/order/src/main/java/store/order/OrderResource.java` | Camada REST. |
| `api/order/src/main/java/store/order/OrderService.java` | Regras de negócio e cálculo dos totais. |
| `api/order/src/main/java/store/order/ProductClient.java` | Cliente Feign para o `product-service`. |
| `api/order/src/main/java/store/order/OrderRepository.java` | Acesso ao banco. |
| `api/order/src/main/java/store/order/OrderModel.java` | Entidade JPA do pedido. |
| `api/order/src/main/java/store/order/OrderStatus.java` | Status do pedido. |
| `api/order/src/main/java/store/order/OrderItemModel.java` | Entidade JPA dos itens. |
| `api/order/src/main/java/store/order/CreateOrderIn.java` | DTO de entrada do pedido. |
| `api/order/src/main/java/store/order/CreateOrderItemIn.java` | DTO de entrada dos itens. |
| `api/order/src/main/resources/db/migration/V1__create_orders.sql` | Migração do banco. |
| `api/order/src/main/resources/db/migration/V2__add_status_to_orders.sql` | Evolução com status do pedido. |
| `api/order/src/main/resources/application.yaml` | Configuração do serviço. |
| `api/order/Jenkinsfile` | Pipeline de build, push da imagem e deploy no EKS. |
| `api/order/k8s/deployment.yaml` | Deployment Kubernetes do serviço. |
| `api/order/k8s/service.yaml` | Service interno `order-service`. |

## Deploy e CI/CD

O pipeline do `order-service` gera a imagem Docker `iquenavarro/order-service`, publica no Docker Hub e aplica os manifests no cluster EKS `eks-store`. No Kubernetes, o serviço recebe a variável `PRODUCT_SERVICE_URL` com o valor `http://product-service:8080`, mantendo a comunicação interna com o catálogo de produtos.

## Trecho de código principal

```java
for (CreateOrderItemIn itemIn : in.items()) {
    ProductSnapshotOut product = fetchProduct(itemIn.idProduct().trim());
    if (product.stock() < itemIn.quantity()) {
        throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "Insufficient stock for product: " + product.id());
    }

    BigDecimal unitPrice = scale(product.price());
    BigDecimal lineTotal = unitPrice.multiply(BigDecimal.valueOf(itemIn.quantity()))
        .setScale(2, RoundingMode.HALF_UP);

    OrderItemModel item = new OrderItemModel();
    item.setProductId(product.id());
    item.setQuantity(itemIn.quantity());
    item.setUnitPriceUsd(unitPrice);
    item.setTotalUsd(lineTotal);
    order.addItem(item);

    total = total.add(lineTotal).setScale(2, RoundingMode.HALF_UP);
}
```
