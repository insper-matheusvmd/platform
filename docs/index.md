# Plataforma de Microsserviços

## Escopo do projeto

Este projeto foi realizado por Matheus Vicco e Helio Henrique Navarro. A proposta é uma aplicação web baseada em microsserviços para compra e venda de produtos, aplicando os conteúdos de AWS, EKS, CI/CD, observabilidade, testes de carga e análise de gargalos vistos durante o curso.

A plataforma é composta por serviços independentes para autenticação, contas, catálogo de produtos, pedidos, taxas de câmbio e roteamento via gateway. O projeto principal incorpora os serviços como submódulos Git e centraliza a documentação publicada com MkDocs.

## Serviços principais

| Serviço | Responsabilidade |
|---|---|
| `gateway-service` | Entrada única da aplicação e roteamento para os microsserviços internos. |
| `auth-service` | Login, registro, geração e validação de token JWT. |
| `account-service` | Cadastro e consulta de contas de usuários. |
| `product-service` | Cadastro, consulta, listagem e remoção de produtos, incluindo descrição, preço e estoque. |
| `order-service` | Criação e consulta de pedidos, validação de produtos, verificação de estoque e cálculo de totais. |
| `exchange-service` | Consulta de taxas de câmbio entre moedas para apoiar transações em diferentes moedas. |

## Infraestrutura e entrega

| Item | Descrição |
|---|---|
| Repositório | [insper-matheusvmd/platform](https://github.com/insper-matheusvmd/platform) |
| Documentação publicada | [GitHub Pages](https://insper-matheusvmd.github.io/platform/) |
| Execução local | Docker Compose para os serviços e infraestrutura de apoio. |
| Deploy | Manifests Kubernetes e pipelines Jenkins para publicação no EKS. |
| Observabilidade | Actuator, Prometheus e Grafana. |
| Teste de carga | HPA no `gateway-service` com geração de carga dentro do cluster. |
