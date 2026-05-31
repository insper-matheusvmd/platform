# Código Fonte das Atividades

## Product Service

Código no repositório:

- `api/product/pom.xml`
- `api/product/Dockerfile`
- `api/product/Jenkinsfile`
- `api/product/src/main/java/store/product/ProductApplication.java`
- `api/product/src/main/java/store/product/ProductResource.java`
- `api/product/src/main/java/store/product/ProductService.java`
- `api/product/src/main/java/store/product/ProductRepository.java`
- `api/product/src/main/java/store/product/ProductModel.java`
- `api/product/src/main/java/store/product/ProductIn.java`
- `api/product/src/main/java/store/product/ProductOut.java`
- `api/product/src/main/java/store/product/ApiError.java`
- `api/product/src/main/java/store/product/ApiExceptionHandler.java`
- `api/product/src/main/resources/application.yaml`
- `api/product/src/main/resources/db/migration/V1__create_products.sql`
- `api/product/src/main/resources/db/migration/V2__add_description_and_stock_to_products.sql`

## Order Service

Código no repositório:

- `api/order/pom.xml`
- `api/order/Dockerfile`
- `api/order/Jenkinsfile`
- `api/order/src/main/java/store/order/OrderApplication.java`
- `api/order/src/main/java/store/order/OrderResource.java`
- `api/order/src/main/java/store/order/OrderService.java`
- `api/order/src/main/java/store/order/ProductClient.java`
- `api/order/src/main/java/store/order/OrderRepository.java`
- `api/order/src/main/java/store/order/OrderModel.java`
- `api/order/src/main/java/store/order/OrderItemModel.java`
- `api/order/src/main/java/store/order/OrderStatus.java`
- `api/order/src/main/java/store/order/CreateOrderIn.java`
- `api/order/src/main/java/store/order/CreateOrderItemIn.java`
- `api/order/src/main/java/store/order/OrderCreatedOut.java`
- `api/order/src/main/java/store/order/OrderDetailsOut.java`
- `api/order/src/main/java/store/order/OrderSummaryOut.java`
- `api/order/src/main/java/store/order/OrderItemOut.java`
- `api/order/src/main/java/store/order/ProductRefOut.java`
- `api/order/src/main/java/store/order/ProductSnapshotOut.java`
- `api/order/src/main/java/store/order/ApiError.java`
- `api/order/src/main/java/store/order/ApiExceptionHandler.java`
- `api/order/src/main/resources/application.yaml`
- `api/order/src/main/resources/db/migration/V1__create_orders.sql`
- `api/order/src/main/resources/db/migration/V2__add_status_to_orders.sql`

## Exchange Service

Código no repositório:

- `api/exchange-service/main.py`
- `api/exchange-service/requirements.txt`
- `api/exchange-service/Dockerfile`
- `api/exchange-service/Jenkinsfile`
- `api/exchange-service/k8s/deployment.yaml`
- `api/exchange-service/k8s/service.yaml`
