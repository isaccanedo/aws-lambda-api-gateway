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

# 3. Dependências
Precisaremos das mesmas dependências do artigo AWS Lambda Usando DynamoDB com Java.

Além disso, também precisamos da biblioteca JSON Simple:

```
<dependency>
    <groupId>com.googlecode.json-simple</groupId>
    <artifactId>json-simple</artifactId>
    <version>1.1.1</version>
</dependency>
```
4. Desenvolvimento e implantação das funções Lambda
Nesta seção, vamos desenvolver e construir nossas funções Lambda em Java, vamos implantá-lo usando o AWS Console e vamos executar um teste rápido.

Como queremos demonstrar os recursos básicos de integração do API Gateway com Lambda, criaremos duas funções:

```
Função 1: recebe uma carga útil da API, usando um método PUT
Função 2: demonstra como usar um parâmetro de caminho HTTP ou parâmetro de consulta HTTP proveniente da API
```

Em termos de implementação, criaremos uma classe RequestHandler, que possui dois métodos - um para cada função.

### 4.1. Modelo
Antes de implementar o manipulador de solicitação real, vamos dar uma olhada rápida em nosso modelo de dados:

```
public class Person {

    private int id;
    private String name;

    public Person(String json) {
        Gson gson = new Gson();
        Person request = gson.fromJson(json, Person.class);
        this.id = request.getId();
        this.name = request.getName();
    }

    public String toString() {
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        return gson.toJson(this);
    }

    // getters and setters
}
```

Nosso modelo consiste em uma classe Person simples, que possui duas propriedades. A única parte notável é o construtor Person (String), que aceita uma String JSON.

### 4.2. Implementação da classe RequestHandler
Assim como no artigo AWS Lambda With Java, criaremos uma implementação da interface RequestStreamHandler:

```
public class APIDemoHandler implements RequestStreamHandler {

    private static final String DYNAMODB_TABLE_NAME = System.getenv("TABLE_NAME"); 
    
    @Override
    public void handleRequest(
      InputStream inputStream, OutputStream outputStream, Context context)
      throws IOException {

        // implementation
    }

    public void handleGetByParam(
      InputStream inputStream, OutputStream outputStream, Context context)
      throws IOException {

        // implementation
    }
}
```

Como podemos ver, a interface RequestStreamHander define apenas um método, handeRequest (). De qualquer forma, podemos definir outras funções na mesma classe, como fizemos aqui. Outra opção seria criar uma implementação de RequestStreamHander para cada função.

Em nosso caso específico, escolhemos o primeiro pela simplicidade. No entanto, a escolha deve ser feita caso a caso, levando em consideração fatores como desempenho e capacidade de manutenção do código.

Também lemos o nome de nossa tabela DynamoDB na variável de ambiente TABLE_NAME. Definiremos essa variável posteriormente durante a implantação.

### 4.3. Implementação da Função 1
Em nossa primeira função, queremos demonstrar como obter uma carga útil (como de uma solicitação PUT ou POST) do API Gateway:

```
public void handleRequest(
  InputStream inputStream, 
  OutputStream outputStream, 
  Context context)
  throws IOException {

    JSONParser parser = new JSONParser();
    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
    JSONObject responseJson = new JSONObject();

    AmazonDynamoDB client = AmazonDynamoDBClientBuilder.defaultClient();
    DynamoDB dynamoDb = new DynamoDB(client);

    try {
        JSONObject event = (JSONObject) parser.parse(reader);

        if (event.get("body") != null) {
            Person person = new Person((String) event.get("body"));

            dynamoDb.getTable(DYNAMODB_TABLE_NAME)
              .putItem(new PutItemSpec().withItem(new Item().withNumber("id", person.getId())
                .withString("name", person.getName())));
        }

        JSONObject responseBody = new JSONObject();
        responseBody.put("message", "New item created");

        JSONObject headerJson = new JSONObject();
        headerJson.put("x-custom-header", "my custom header value");

        responseJson.put("statusCode", 200);
        responseJson.put("headers", headerJson);
        responseJson.put("body", responseBody.toString());

    } catch (ParseException pex) {
        responseJson.put("statusCode", 400);
        responseJson.put("exception", pex);
    }

    OutputStreamWriter writer = new OutputStreamWriter(outputStream, "UTF-8");
    writer.write(responseJson.toString());
    writer.close();
}
```

Conforme discutido antes, configuraremos a API posteriormente para usar a integração de proxy Lambda. Esperamos que o API Gateway passe a solicitação completa para a função Lambda no parâmetro InputStream.

Tudo o que precisamos fazer é escolher os atributos relevantes da estrutura JSON contida.

Como podemos ver, o método consiste basicamente em três etapas:

- Buscando o objeto body de nosso fluxo de entrada e criando um objeto Person a partir desse;
- Armazenar esse objeto Person em uma tabela DynamoDB;
- Construir um objeto JSON, que pode conter vários atributos, como um corpo para a resposta, cabeçalhos personalizados, bem como um código de status HTTP.

Um ponto que vale a pena mencionar aqui: o API Gateway espera que o corpo seja uma String (para solicitação e resposta).

Como esperamos obter uma String como corpo do API Gateway, lançamos o corpo em String e inicializamos nosso objeto Person:

```
Person person = new Person((String) event.get("body"));
```

O API Gateway também espera que o corpo da resposta seja uma String:

```
responseJson.put("body", responseBody.toString());
```

Este tópico não é mencionado explicitamente na documentação oficial. No entanto, se olharmos mais de perto, podemos ver que o atributo body é uma String em ambos os trechos da solicitação e também da resposta.

A vantagem deve ser clara: mesmo que JSON seja o formato entre o API Gateway e a função Lambda, o corpo real pode conter texto simples, JSON, XML ou qualquer outro. É então responsabilidade da função Lambda lidar com o formato corretamente.

Veremos a aparência do corpo de solicitação e resposta mais tarde, quando testarmos nossas funções no Console da AWS.

O mesmo também se aplica às duas funções a seguir.

### 4.4. Implementação da Função 2
Em uma segunda etapa, queremos demonstrar como usar um parâmetro de caminho ou um parâmetro de string de consulta para recuperar um item Person do banco de dados usando seu ID:

```
public void handleGetByParam(
  InputStream inputStream, OutputStream outputStream, Context context)
  throws IOException {

    JSONParser parser = new JSONParser();
    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
    JSONObject responseJson = new JSONObject();

    AmazonDynamoDB client = AmazonDynamoDBClientBuilder.defaultClient();
    DynamoDB dynamoDb = new DynamoDB(client);

    Item result = null;
    try {
        JSONObject event = (JSONObject) parser.parse(reader);
        JSONObject responseBody = new JSONObject();

        if (event.get("pathParameters") != null) {
            JSONObject pps = (JSONObject) event.get("pathParameters");
            if (pps.get("id") != null) {
                int id = Integer.parseInt((String) pps.get("id"));
                result = dynamoDb.getTable(DYNAMODB_TABLE_NAME).getItem("id", id);
            }
        } else if (event.get("queryStringParameters") != null) {
            JSONObject qps = (JSONObject) event.get("queryStringParameters");
            if (qps.get("id") != null) {

                int id = Integer.parseInt((String) qps.get("id"));
                result = dynamoDb.getTable(DYNAMODB_TABLE_NAME)
                  .getItem("id", id);
            }
        }
        if (result != null) {
            Person person = new Person(result.toJSON());
            responseBody.put("Person", person);
            responseJson.put("statusCode", 200);
        } else {
            responseBody.put("message", "No item found");
            responseJson.put("statusCode", 404);
        }

        JSONObject headerJson = new JSONObject();
        headerJson.put("x-custom-header", "my custom header value");

        responseJson.put("headers", headerJson);
        responseJson.put("body", responseBody.toString());

    } catch (ParseException pex) {
        responseJson.put("statusCode", 400);
        responseJson.put("exception", pex);
    }

    OutputStreamWriter writer = new OutputStreamWriter(outputStream, "UTF-8");
    writer.write(responseJson.toString());
    writer.close();
}
```
Novamente, três etapas são relevantes:

- Verificamos se um pathParameters ou um array queryStringParameters com um atributo id estão presentes;
- Se verdadeiro, usamos o valor pertencente para solicitar um item de pessoa com esse ID do banco de dados;
- Adicionamos uma representação JSON do item recebido à resposta.
A documentação oficial fornece uma explicação mais detalhada do formato de entrada e formato de saída para integração de proxy.

### 4.5. Código de construção
Novamente, podemos simplesmente construir nosso código usando Maven:

```
mvn clean package shade:shade
```

O arquivo JAR será criado na pasta de destino.

### 4.6. Criação da tabela DynamoDB
Podemos criar a tabela conforme explicado em AWS Lambda usando DynamoDB com Java.

Vamos escolher Pessoa como nome da tabela, id como nome da chave primária e Número como tipo da chave primária.

### 4.7. Implantando código por meio do console AWS
Depois de construir nosso código e criar a tabela, podemos agora criar as funções e fazer o upload do código.

Isso pode ser feito repetindo as etapas 1 a 5 do artigo AWS Lambda com Java, uma vez para cada um de nossos dois métodos.

Vamos usar os seguintes nomes de função:

- StorePersonFunction para o método handleRequest (função 1);
- GetPersonByHTTPParamFunction para o método handleGetByParam (função 2).
Também temos que definir uma variável de ambiente TABLE_NAME com o valor “Person”.

### 4.8. Testando as funções
Antes de continuar com a parte real do API Gateway, podemos executar um teste rápido no AWS Console, apenas para verificar se nossas funções do Lambda estão funcionando corretamente e podem lidar com o formato de integração de proxy.

O teste de uma função Lambda no AWS Console funciona conforme descrito no artigo AWS Lambda com Java.

No entanto, quando criamos um evento de teste, temos que considerar o formato especial de integração de proxy, que nossas funções esperam. Podemos usar o modelo API Gateway AWS Proxy e personalizá-lo para nossas necessidades ou podemos copiar e colar os seguintes eventos:

Para StorePersonFunction, devemos usar isto:

```
{
    "body": "{\"id\": 1, \"name\": \"John Doe\"}"
}
```
Conforme discutido anteriormente, o corpo deve ter o tipo String, mesmo que contenha uma estrutura JSON. O motivo é que o API Gateway enviará suas solicitações no mesmo formato.

A seguinte resposta deve ser retornada:

```
{
    "isBase64Encoded": false,
    "headers": {
        "x-custom-header": "my custom header value"
    },
    "body": "{\"message\":\"New item created\"}",
    "statusCode": 200
}
```

Aqui, podemos ver que o corpo de nossa resposta é uma String, embora contenha uma estrutura JSON.

Vejamos a entrada para GetPersonByHTTPParamFunction.

Para testar a funcionalidade do parâmetro de caminho, a entrada ficaria assim:

```
{
    "pathParameters": {
        "id": "1"
    }
}
```

E a entrada para enviar um parâmetro de string de consulta seria:

```
{
    "queryStringParameters": {
        "id": "1"
    }
}
```

Como resposta, devemos obter o seguinte para os métodos de ambos os casos:

```
{
  "headers": {
    "x-custom-header": "my custom header value"
  },
  "body": "{\"Person\":{\n  \"id\": 88,\n  \"name\": \"John Doe\"\n}}",
  "statusCode": 200
}
```

Novamente, o corpo é uma corda.

# 5. Criação e teste da API
Depois de criar e implantar as funções do Lambda na seção anterior, agora podemos criar a API real usando o AWS Console.

Vejamos o fluxo de trabalho básico:

- Crie uma API em nossa conta AWS;
- Adicione um recurso à hierarquia de recursos da API;
- Crie um ou mais métodos para o recurso;
- Configure a integração entre um método e a função Lambda pertencente.
Repetiremos as etapas 2 a 4 para cada uma de nossas duas funções nas seções a seguir.

### 5.1. Criação da API
Para criar a API, teremos que:

Faça login no console do API Gateway em https://console.aws.amazon.com/apigateway
Clique em “Primeiros passos” e selecione “Nova API”
Digite o nome da nossa API (TestAPI) e confirme clicando em “Criar API”
Tendo criado a API, agora podemos criar a estrutura da API e vinculá-la às nossas funções Lambda.

### 5.2 Estrutura API para Função 1

As etapas a seguir são necessárias para nosso StorePersonFunction:

#### 1 - Escolha o item de recurso pai na árvore “Recursos” e selecione “Criar Recurso” no menu suspenso “Ações”. Então, temos que fazer o seguinte no painel “Novo recurso filho”:
- Digite “Pessoas” como um nome no campo de texto de entrada “Nome do Recurso”
- Deixe o valor padrão no campo de texto de entrada “Caminho do recurso”
- Escolha “Criar Recurso”.

#### 2 - Escolha o recurso recém-criado, escolha “Criar Método” no menu suspenso “Ações” e siga os seguintes passos:
- Escolha PUT na lista suspensa do método HTTP e, em seguida, escolha o ícone de marca de seleção para salvar a escolha
- Deixe “Função Lambda” como tipo de integração e selecione a opção “Usar integração Lambda Proxy”.
- Escolha a região de “Região Lambda”, onde implantamos nossas funções Lambda antes
- Digite “StorePersonFunction” em “Lambda Function”

#### 3. Escolha "Salvar" e confirme com "OK" quando solicitado com "Adicionar permissão à função Lambda"

5.3. Estrutura da API para a função 2 - parâmetros de caminho
As etapas para recuperar nossos parâmetros de caminho são semelhantes:

Escolha o item de recurso / people na árvore “Recursos” e, a seguir, selecione “Criar Recurso” no menu suspenso “Ações”. Então, temos que fazer o seguinte no painel Novo recurso filho:
Digite “Pessoa” como um nome no campo de texto de entrada “Nome do recurso”
Altere o campo de texto de entrada “Caminho do recurso” para “{id}”
Escolha “Criar Recurso”
Escolha o recurso recém-criado, selecione “Criar Método” no menu suspenso “Ações” e execute as seguintes etapas:
Escolha GET na lista suspensa do método HTTP e, em seguida, escolha o ícone de marca de seleção para salvar a escolha
Deixe “Função Lambda” como tipo de integração e selecione a opção “Usar integração Lambda Proxy”.
Escolha a região de “Região Lambda”, onde implantamos nossas funções Lambda antes
Digite “GetPersonByHTTPParamFunction” em “Função Lambda”
Escolha "Salvar" e confirme com "OK" quando solicitado com "Adicionar permissão à função Lambda"
Nota: é importante aqui definir o parâmetro “Resource Path” para “{id}”, pois nosso GetPersonByPathParamFunction espera que este parâmetro seja nomeado exatamente assim.

5,4 Estrutura da API para a função 2 - Parâmetros da string de consulta
As etapas para receber parâmetros de string de consulta são um pouco diferentes, pois não precisamos criar um recurso, mas sim um parâmetro de consulta para o parâmetro id:

Escolha o item de recurso / people na árvore "Recursos", selecione "Criar Método" no menu suspenso "Ações" e execute as seguintes etapas:
Escolha GET na lista suspensa do método HTTP e selecione o ícone da marca de seleção para salvar a escolha
Deixe “Função Lambda” como tipo de integração e selecione a opção “Usar integração Lambda Proxy”.
Escolha a região de “Região Lambda”, onde implantamos nossas funções Lambda antes
Digite “GetPersonByHTTPParamFunction” em “Função Lambda”.
Escolha "Salvar" e confirme com "OK" quando solicitado com "Adicionar permissão à função Lambda"
Escolha “Solicitação de método” à direita e execute as seguintes etapas:
Expanda a lista de parâmetros de string de consulta de URL
Clique em “Add Query String”
Digite “id” no campo do nome e escolha o ícone da marca de seleção para salvar
Selecione a caixa de seleção “Obrigatório”
Clique no símbolo de caneta ao lado de “Solicitar validador” na parte superior do painel, selecione “Validar parâmetros e cabeçalhos de string de consulta” e escolha o ícone de marca de seleção
Nota: É importante definir o parâmetro “Query String” para “id”, pois nosso GetPersonByHTTPParamFunction espera que este parâmetro seja nomeado exatamente assim.

### 5.5. Testando a API
Nossa API já está pronta, mas ainda não é pública. Antes de publicá-lo, queremos primeiro executar um teste rápido no Console.

Para isso, podemos selecionar o respectivo método a ser testado na árvore “Recursos” e clicar no botão “Testar”. Na tela seguinte, podemos digitar nossa entrada, como faríamos com um cliente via HTTP.

Para StorePersonFunction, temos que digitar a seguinte estrutura no campo “Request Body”:

```
{
    "id": 2,
    "name": "Jane Doe"
}
```

Para GetPersonByHTTPParamFunction com parâmetros de caminho, temos que digitar 2 como um valor no campo “{id}” em “Caminho”.

Para GetPersonByHTTPParamFunction com parâmetros de string de consulta, temos que digitar id = 2 como um valor no campo “{people}” em “Query Strings”.

### 5.6. Implantar a API
Até agora, nossa API não era pública e, portanto, estava disponível apenas no AWS Console.

Conforme discutido antes, quando implantamos uma API, temos que associá-la a um estágio, que é como um instantâneo no tempo da API. Se reimplantarmos uma API, podemos atualizar um estágio existente ou criar um novo.

Vamos ver como ficará o esquema de URL de nossa API:

```
https://{restapi-id}.execute-api.{region}.amazonaws.com/{stageName}
```

As seguintes etapas são necessárias para a implantação:

- Escolha a API específica no painel de navegação “APIs”
- Escolha “Ações” no painel de navegação Recursos e selecione “Implementar API” no menu suspenso “Ações”
- Escolha “[Novo Estágio]” na lista suspensa “Estágio de implantação”, digite “teste” em “Nome do estágio” e, opcionalmente, forneça uma descrição do estágio e implantação
- Acione a implantação escolhendo “Implantar”
Após a última etapa, o console fornecerá a URL raiz da API, por exemplo, https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test.

### 5.7. Invocando o Endpoint
Como a API é pública agora, podemos chamá-la usando qualquer cliente HTTP que quisermos.

Com cURL, as chamadas seriam as seguintes.

StorePersonFunction:

```
curl -X PUT 'https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test/persons' \
  -H 'content-type: application/json' \
  -d '{"id": 3, "name": "Richard Roe"}'
```

GetPersonByHTTPParamFunction para parâmetros de caminho:

```
curl -X GET 'https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test/persons/3' \
  -H 'content-type: application/json'
```

GetPersonByHTTPParamFunction para parâmetros de string de consulta:

```
curl -X GET 'https://0skaqfgdw4.execute-api.eu-central-1.amazonaws.com/test/persons?id=3' \
  -H 'content-type: application/json'
```

# 6. Conclusão
Neste artigo, vimos como tornar as funções do AWS Lambda disponíveis como endpoints REST, usando o AWS API Gateway.

Exploramos os conceitos básicos e a terminologia do API Gateway e aprendemos como integrar funções Lambda usando Lambda Proxy Integration.

Por fim, vimos como criar, implantar e testar uma API.
