# Atividades Realizadas

## Visão geral

As atividades individuais foram concentradas em dois microsserviços da plataforma:

| Serviço | Responsabilidade |
|---|---|
| `product-service` | Manter o catálogo de produtos, incluindo criação, listagem, busca por ID e remoção. |
| `order-service` | Registrar pedidos de uma conta, consultar pedidos e integrar com produtos para validar itens. |

## Atividade 1 - Product Service

O serviço de produtos foi implementado com Spring Boot, Spring MVC, Spring Data JPA, Flyway e PostgreSQL. Ele expõe endpoints REST em `/products` e persiste os dados no schema `products`.

Principais entregas:

- API REST para cadastro, listagem, consulta e remoção de produtos.
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
- Snapshot de preço unitário e total em USD.
- Persistência de pedidos e itens em tabelas separadas.
- Endpoint Prometheus via Spring Actuator.

Detalhes: [Order Service](../modulos/order.md).

