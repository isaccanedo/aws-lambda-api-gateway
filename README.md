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
