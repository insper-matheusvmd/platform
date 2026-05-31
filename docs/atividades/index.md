# Atividades Realizadas

## Visão geral

As atividades documentadas cobrem os microsserviços principais da plataforma:

| Serviço | Responsabilidade |
|---|---|
| `product-service` | Manter o catálogo de produtos, incluindo criação, listagem, busca por ID, remoção, descrição, preço e estoque. |
| `order-service` | Registrar pedidos de uma conta, consultar pedidos, manter status e integrar com produtos para validar itens e estoque. |
| `exchange-service` | Consultar taxas de câmbio entre moedas usando uma API externa e expor o resultado para a plataforma. |

## Atividade 1 - Product Service

O serviço de produtos foi implementado com Spring Boot, Spring MVC, Spring Data JPA, Flyway e PostgreSQL. Ele expõe endpoints REST em `/products` e persiste os dados no schema `products`.

Principais entregas:

- API REST para cadastro, listagem, consulta e remoção de produtos.
- Campos de nome, descrição, preço, estoque e unidade.
- Validação de entrada com Bean Validation.
- Persistência relacional com JPA.
- Migração de banco com Flyway.
- Endpoint Prometheus via Spring Actuator.

Detalhes: [Product Service](../modulos/product.md).

## Atividade 2 - Order Service

O serviço de pedidos foi implementado com Spring Boot, Spring MVC, Spring Data JPA, Flyway, PostgreSQL e OpenFeign. Ele expõe endpoints REST em `/orders` e consome o `product-service` antes de gravar um pedido.

Principais entregas:

- API REST para criação, listagem e consulta de pedidos.
- Separação entre resumo e detalhe do pedido.
- Validação de produtos via chamada HTTP interna.
- Validação de estoque disponível antes da criação do pedido.
- Snapshot de preço unitário e total em USD.
- Status inicial `CREATED` para pedidos criados.
- Persistência de pedidos e itens em tabelas separadas.
- Endpoint Prometheus via Spring Actuator.

Detalhes: [Order Service](../modulos/order.md).

## Atividade 3 - Exchange Service

O serviço de câmbio foi implementado com FastAPI e consulta a API externa AwesomeAPI para obter a cotação entre duas moedas. Ele expõe endpoints em `/exchanges` e retorna valores de compra, venda e data da cotação.

Principais entregas:

- API REST para consulta de câmbio entre duas moedas.
- Normalização das moedas para letras maiúsculas.
- Integração HTTP externa com AwesomeAPI.
- Tratamento de erro para falha de integração e par de moedas inexistente.
- Deploy via Docker, Jenkins e manifests Kubernetes.

Detalhes: [Exchange Service](../modulos/exchange.md).
