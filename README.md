# aws-lambda-api-gateway
Usando AWS Lambda com API Gateway

# 1. Visão Geral
O AWS Lambda é um serviço de computação sem servidor fornecido pela Amazon Web Services.

Em dois artigos anteriores, discutimos como criar uma função AWS Lambda usando Java, bem como acessar o DynamoDB a partir de uma função Lambda.

Neste tutorial, discutiremos como publicar uma função Lambda como um endpoint REST, usando o AWS Gateway.

Teremos uma visão detalhada dos seguintes tópicos:

- Conceitos e termos básicos do API Gateway
- Integração de funções Lambda com API Gateway usando integração Lambda Proxy
- Criação de uma API, sua estrutura e como mapear os recursos da API nas funções Lambda
- Implantação e teste da API

# 2. Noções básicas e termos
API Gateway é um serviço totalmente gerenciado que permite aos desenvolvedores criar, publicar, manter, monitorar e proteger APIs em qualquer escala.

Podemos implementar uma interface de programação baseada em HTTP consistente e escalonável (também conhecida como serviços RESTful)
para acessar serviços de back-end como funções Lambda, outros serviços AWS (por exemplo, EC2, S3, DynamoDB) e quaisquer endpoints HTTP.

Os recursos incluem, mas não estão limitados a:

- Gestão de tráfego;
- Autorização e controle de acesso;
- Monitoramento;
- Gerenciamento de versão de API;
- Limitação de solicitações para evitar ataques.

Como o AWS Lambda, o API Gateway é dimensionado automaticamente e é cobrado por chamada API.

Informações detalhadas podem ser encontradas na documentação oficial.

2.1. Termos
API Gateway é um serviço da AWS que oferece suporte à criação, implantação e gerenciamento de uma interface de programação de aplicativo RESTful para expor endpoints HTTP de back-end, funções do AWS Lambda e outros serviços da AWS.

Uma API Gateway API é uma coleção de recursos e métodos que podem ser integrados com funções Lambda, outros serviços AWS ou endpoints HTTP no back-end. A API consiste em recursos que formam a estrutura da API. Cada recurso de API pode expor um ou mais métodos de API que devem ter verbos HTTP exclusivos.

Para publicar uma API, temos que criar uma implementação de API e associá-la a um assim chamado estágio. Um estágio é como um instantâneo no tempo da API. Se reimplantarmos uma API, podemos atualizar um estágio existente ou criar um novo. Com isso, diferentes versões de uma API ao mesmo tempo são possíveis, por exemplo, um estágio de desenvolvimento, um estágio de teste e até mesmo várias versões de produção, como v1, v2, etc.

A integração do Lambda Proxy é uma configuração simplificada para a integração entre as funções do Lambda e o API Gateway.

O API Gateway envia a solicitação inteira como uma entrada para uma função Lambda de back-end. Em termos de resposta, o API Gateway transforma a saída da função Lambda de volta em uma resposta HTTP de front-end.
