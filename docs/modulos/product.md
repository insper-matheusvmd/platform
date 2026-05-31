# Product Service

## Objetivo

O `product-service` centraliza o catálogo de produtos da aplicação. Ele guarda nome, descrição, preço, estoque e unidade de cada produto e oferece uma API para que outros serviços consultem esses dados.

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
  "description": "Café torrado em grãos",
  "price": 12.90,
  "stock": 30,
  "unit": "pacote"
}
```

Regras de validação:

- `name` é obrigatório.
- `description` é obrigatório.
- `price` é obrigatório e deve ser maior ou igual a `0.01`.
- `stock` deve ser maior ou igual a `0`.
- `unit` é obrigatório.

## Saída

```json
{
  "id": "uuid-do-produto",
  "name": "Café",
  "description": "Café torrado em grãos",
  "price": 12.90,
  "stock": 30,
  "unit": "pacote"
}
```

## Persistência

O serviço usa o schema `products` e a tabela `products.products`.

```sql
CREATE TABLE IF NOT EXISTS products.products (
    id VARCHAR(36) PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description VARCHAR(1024) NOT NULL,
    price NUMERIC(12, 2) NOT NULL,
    stock INTEGER NOT NULL,
    unit VARCHAR(64) NOT NULL
);
```

A evolução `V2__add_description_and_stock_to_products.sql` adiciona `description` e `stock` em bancos já existentes.

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
| `api/product/src/main/resources/db/migration/V2__add_description_and_stock_to_products.sql` | Evolução com descrição e estoque. |
| `api/product/src/main/resources/application.yaml` | Configuração do serviço. |
| `api/product/Jenkinsfile` | Pipeline de build, push da imagem e deploy no EKS. |
| `api/product/k8s/deployment.yaml` | Deployment Kubernetes do serviço. |
| `api/product/k8s/service.yaml` | Service interno `product-service`. |

## Deploy e CI/CD

O pipeline do `product-service` gera a imagem Docker `iquenavarro/product-service`, publica no Docker Hub e aplica os manifests no cluster EKS `eks-store`. O Deployment usa o nome `product-service`, permitindo que outros serviços internos, como o `order-service`, acessem a API por `http://product-service:8080`.

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
    model.setDescription(in.description().trim());
    model.setUnit(in.unit().trim());
    model.setPrice(scale(in.price()));
    model.setStock(in.stock());
    return toOut(repository.save(model));
}
```
