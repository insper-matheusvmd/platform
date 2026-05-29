# Bottlenecks Implementados

## Bottleneck 1 - Validação síncrona de produto no pedido

O `order-service` depende do `product-service` para criar um pedido. Para cada item recebido, o serviço chama `GET /products/{id}` antes de persistir o pedido.

Essa decisão cria um bottleneck intencional: a criação do pedido fica limitada pela disponibilidade e latência do serviço de produtos. Em troca, o pedido nunca é criado com um produto inexistente e sempre registra o preço retornado pelo catálogo.

Arquivo principal:

- `api/order/src/main/java/store/order/ProductClient.java`
- `api/order/src/main/java/store/order/OrderService.java`

Trecho:

```java
ProductSnapshotOut product = fetchProduct(itemIn.idProduct().trim());
BigDecimal unitPrice = scale(product.price());
```

Tratamento de falhas:

```java
catch (FeignException.NotFound ex) {
    throw new ResponseStatusException(HttpStatus.BAD_REQUEST, "Product not found: " + idProduct);
}
```

## Bottleneck 2 - Persistência transacional de pedidos e itens

A criação de pedidos é feita dentro de uma transação. O pedido só é salvo depois que todos os produtos foram validados, todos os itens foram montados e o total foi calculado.

Esse ponto também funciona como bottleneck: o banco de dados precisa confirmar a gravação do pedido e dos itens juntos. A vantagem é manter consistência entre cabeçalho do pedido, itens e total.

Arquivo principal:

- `api/order/src/main/java/store/order/OrderService.java`
- `api/order/src/main/resources/db/migration/V1__create_orders.sql`

Trecho:

```java
@Transactional
public OrderCreatedOut create(String accountId, CreateOrderIn in) {
    OrderModel order = new OrderModel();
    order.setAccountId(accountId);
    order.setCreatedAt(LocalDateTime.now());
    ...
    OrderModel saved = orderRepository.saveAndFlush(order);
    return toCreatedOut(saved);
}
```

## Bottleneck 3 - Ordenação da listagem de produtos

A listagem de produtos usa ordenação por nome direto na consulta ao banco. Isso facilita a leitura para o usuário, mas desloca trabalho para a camada de persistência.

Arquivo principal:

- `api/product/src/main/java/store/product/ProductService.java`

Trecho:

```java
return repository.findAll(Sort.by(Sort.Direction.ASC, "name"))
    .stream()
    .map(this::toOut)
    .toList();
```

