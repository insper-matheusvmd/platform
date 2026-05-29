# Product Service

## Objetivo

O `product-service` centraliza o catálogo de produtos da aplicação. Ele guarda nome, preço e unidade de cada produto e oferece uma API para que outros serviços consultem esses dados.

## Tecnologias

| Tecnologia | Uso |
|---|---|
| Spring Boot | Aplicação principal |
| Spring MVC | Endpoints REST |
| Spring Data JPA | Persistência |
| PostgreSQL | Banco de dados |
| Flyway | Migrações |
| Bean Validation | Validação de entrada |
| Actuator + Prometheus | Métricas |

## Endpoints

| Método | Rota | Descrição |
|---|---|---|
| `POST` | `/products` | Cria um produto. |
| `GET` | `/products` | Lista produtos em ordem alfabética. |
| `GET` | `/products/{id}` | Busca um produto por ID. |
| `DELETE` | `/products/{id}` | Remove um produto por ID. |
| `GET` | `/products/actuator/prometheus` | Exposição de métricas. |

## Entrada de criação

```json
{
  "name": "Café",
  "price": 12.90,
  "unit": "pacote"
}
```

Regras de validação:

- `name` é obrigatório.
- `price` é obrigatório e deve ser maior ou igual a `0.01`.
- `unit` é obrigatório.

## Saída

```json
{
  "id": "uuid-do-produto",
  "name": "Café",
  "price": 12.90,
  "unit": "pacote"
}
```

## Persistência

O serviço usa o schema `products` e a tabela `products.products`.

```sql
CREATE TABLE IF NOT EXISTS products.products (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    price NUMERIC(12, 2) NOT NULL,
    unit VARCHAR(64) NOT NULL
);
```

## Arquivos principais

| Arquivo | Função |
|---|---|
| `api/product/src/main/java/store/product/ProductApplication.java` | Inicialização da aplicação. |
| `api/product/src/main/java/store/product/ProductResource.java` | Camada REST. |
| `api/product/src/main/java/store/product/ProductService.java` | Regras de negócio. |
| `api/product/src/main/java/store/product/ProductRepository.java` | Acesso ao banco. |
| `api/product/src/main/java/store/product/ProductModel.java` | Entidade JPA. |
| `api/product/src/main/java/store/product/ProductIn.java` | DTO de entrada. |
| `api/product/src/main/java/store/product/ProductOut.java` | DTO de saída. |
| `api/product/src/main/resources/db/migration/V1__create_products.sql` | Migração do banco. |
| `api/product/src/main/resources/application.yaml` | Configuração do serviço. |

## Trechos de código

### Controller REST

```java
@RestController
@Validated
@RequestMapping("/products")
public class ProductResource {
    @PostMapping
    public ResponseEntity<ProductOut> create(@Valid @RequestBody ProductIn in) {
        ProductOut out = service.create(in);
        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(out.id())
            .toUri();
        return ResponseEntity.created(location).body(out);
    }
}
```

### Serviço

```java
public ProductOut create(ProductIn in) {
    ProductModel model = new ProductModel();
    model.setName(in.name().trim());
    model.setUnit(in.unit().trim());
    model.setPrice(scale(in.price()));
    return toOut(repository.save(model));
}
```

