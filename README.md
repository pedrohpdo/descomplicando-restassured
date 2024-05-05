<div align="center">

  ![Java](https://img.shields.io/badge/Java-ED8B00?style=for-the-badge&logo=coffeescript&logoColor=white)&nbsp;
  ![RestAssured](https://img.shields.io/badge/RestAssured-4EA94B?style=for-the-badge&logo=globe&logoColor=white)&nbsp;

</div>

# Descomplicando Restassured
RestAssured, nada mais é do que uma biblioteca em Java para testar e validar API`s RESTful. Simplificando o processo de testes, ela oferece uma linguagem própria, tornando os testes mais fáceis de escrever.

- [Características](#características)
- [Dependências](#depedências)
    -  [Principais](#principais)
    - [Auxiliares](#auxiliares)
- [Given - Then - When](#given---when---then)
- [Primeiro Teste](#primeiro-teste)
- [Testes Dinâmicos](#testes-dinâmicos-massa-de-dados)
    - [Objetos](#utilizando-objetos)
    - [Faker](#faker-dados-dinâmicos)
- [Validações e Asserções](#validações-e-asserções)
    - [Hamcrest](#hamcrest)
    - [JUnit 5](#junit5)

## Características
- Utiliza sintaxe baseada em Gherkin, utilizando palavras chaves que tornam scripts mais legíveis e compreensíveis.
- Permite validação de status de resposta, cabeçalhos e corpo de maneira eficiente, assim como dá suporte a autenticação das requisições, utilizando Basic, OAuth, OAuth 2.0, etc.
- Permite extrair dados de resposta para serem reaproveitados em outras partes do testes, também permite serializar respostas em objetos java.
- Integração ao ecossistema Java: Integra-se perfeitamente com recurso de testes do Java, como **Maven**, **Gradle**, **JUnit** e **TestNG**.

# Depedências
## Principais
```xml
    <dependency>
        <groupId>io.rest-assured</groupId>
        <artifactId>rest-assured</artifactId>
        <version>5.4.0</version>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.0</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.17.0</version>
    </dependency>
    
```
## Auxiliares
```xml
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.22</version>
        <scope>compile</scope>
    </dependency>
    <dependency>
        <groupId>net.datafaker</groupId>
        <artifactId>datafaker</artifactId>
        <version>2.2.2</version>
    </dependency>
```
- **Lombok:** Muito conhecida por quem utiliza java, dinamiza a criação de alguns alguns scripts e os implementa em tempo de execução.
- **Faker:** Utilizamos para geração de dados dinâmicos, o que dinamiza a “mão da massa” na hora de pensar em nomenclaturas de nomes, senhas, cpf’s ou outros tipos de dados. A API do faker provê uma vasta biblioteca que é capaz de randomizar dados e acelerar o processo de desenvolvimento.

# Given - When - Then
A estrutura básica de um teste utilizando RestAssured segue o padrão Given-When-Then, uma
abordagem comum no Desenvolvimento Guiado por Comportamento (Behavior-Driven Development- BDD).

- **Given:** Preparação de ambiente, definição de todas as pré-condições e configuração da sua requisição. Inclusão de cabeçalhos, parâmetros, corpo de requisição, etc.

- **When:** Onde sua request acontece, sua solicitação HTTP é processada e testada.

- **Then:** Aqui é onde você vão acontecer validações e asserções. Mensagens de respostas, status code, cabeçalhos, tempo de resposta ou qualquer outra coisa pode ser validada aqui.

# Primeiro Teste
```java
import org.junit.jupiter.api.Test;
import static io.restassured.RestAssured.given;

@Test
public void testDeveEfetuarRequestComSucesso() {
    given()
        .baseUri("http://localhost:3000")
        .contentType(ContentType.JSON)
        .body("""
            {
            "nome": "Pedro Henrique",
            "email": "pedro@email.com",
            "password" : "pedro123",
            "administrador" : "true"
            }
            """)
    .when()
        .post("/usuario")
    .then()
            .contentType(ContentType.JSON)
            .statusCode(200)
            .body("message", equalTo("Cadastro realizado com sucesso"));
}
```
- **.baseUri():** Propriedade que indica qual a URI que estamos apontando no teste;

- **.contentType():** Indica o tipo de conteúdo que vamos enviar na requisição de teste. Perceba que ele é utilizado tanto na prepação do teste como na validação.

- **.body():** O corpo da requisição, caso seja necessário;

- **.post():** Método HTTP utilizado na requisição (Cada método HTTP tem seu método próprio)

- **.statusCode():** Validar o código HTTP retornado por essa requisição

- **.body():** Nesse momento, vamos fazer validações e asserções no corpo da resposta da requisição.

# Testes Dinâmicos: Massa de Dados
## Utilizando Objetos
```java
import lombok.AllArgsConstructor;
import lombok.NoArgsConstructor;
import lombok.Builder;
import lombok.Data;

@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class UsuarioCreateRequestDTO {
    private String nome;
    private String email;
    private String password;
    private String administrador;
}
```
**Getters e Setters são obrigatórios**

## Faker: Dados dinâmicos
```java
public class UsuarioFactory {
    private static Faker faker = new Faker(new Locale("pt-BR"));
    
    public static UsuarioCreateRequestDTO usuarioComDadosValidos() {
        return UsuarioCreateRequestDTO.builder()
                .nome(faker.name().firstName() + faker.name().lastName())
                .email(faker.internet().emailAddress())
                .password(faker.internet().password())
                .administrador(true)
                .build();
    }

}
```

## Nova Configuração de Teste
```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static io.restassured.RestAssured.baseURI;
import static io.restassured.RestAssured.given;

public class UsuarioFuncionalTest {

    @BeforeEach
    public void setup() {
        baseURI = "http://localhost:3000";
    }
    
    @Test
    public void testCadastroDeUsuarioComDadosValidos() {
        
        given()
            .contentType(ContentType.JSON)
            .body(UsuarioFactory.usuarioComDadosValidos())
        .when()
            .post("/usuario")
        .then()
                .contentType(ContentType.JSON)
                .statusCode(HttpStatus.SC_OK)
                .body("message", equalTo(""));
    }
}

```

# Validações e Asserções
## Hamcrest
Hamcrest é um framework feito para o ecossistema java para asserção de objetos que os permite dar match de uma maneira declarativa. É um recurso flexível e totalmente suportaodo pelo Junit, que nos permite validar algumas situações de maneira mais precisa.

```java
import static org.hamcrest.Matchers.*;

@Test
public void testCadastroProdutoComTokenInvalido() {
    given()
        .baseUri("http://localhost:3000")
        .contentType(ContentType.JSON) 
        .body(requestBody)
    .when()
        .post()
    .then()
        .contentType(ContentType.JSON)
        .statusCode(HttpStatus.SC_UNAUTHORIZED)
        .body("message", notNullValue())
        .body("message", equalTo("Token de acesso ausente, inválido, expirado ou usuário do token não existe mais"));
}
```
Nós o integramos diretamente com escopo do restassured, o que nos provê alguns recursos mais dinamicos para fazer algumas asserções.

No exemplo usado temos:

- **notNullValue():** Valida se o campo “message” vindo como resposta não é um valor nulo
- **equalTo(param):** valida se o campo “mensage” corresponde com a mesangem passada como parametro

## JUnit5

### Extração de Dados do Corpo da Resposta

Vamos imaginar que a resposta da requisição do nosso primeiro teste seja essa
```json
{
	"_id": "oidjas9234",
	"message" : "Cadastro realizado com sucesso"
}
```
### Utilizando Caminho da Resposta: .extract()**.path()**

Utilizando essa abordagem, podemos utilizar um método para extrair um caminho da resposta diretamente para uma variável java

```java
@Test
public void testCadastroDeUsuarioComDadosValidos() {

    String responseMessage = given()
        .baseUri("http://localhost:3000")
        .contentType(ContentType.JSON)
        .body(UsuarioFactory.usuarioComDadosValidos())
    .when()
        .post("/usuario")
    .then()
          .contentType(ContentType.JSON)
          .statusCode(HttpStatus.SC_OK)
          .extract().path("message");
          
    Assertions.assertEquals(responseMessage, "Cadastro realizado com sucesso")
}
```

Aqui mudamos um pouco a dinamica, e agora trabalhamos diretamente com objetos java, de forma que atribuidos a variável **mensagemReposta** ao valor encontrado no path message vindo na nossa resposta.
### Utilizando Objetos Java: .extract().as()

Agora vamos trabalhar com uma solução um pouco mais robusta, vamos trazer o corpo inteiro da resposta para um objeto java, utilizando o mesmo conceito de serializacao e desserialização.

Da mesma forma que fizemos uma classe para o corpo da nossa requisição, vamos implementar um para nossa resposta.

```java
package model.usuario;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class UsuarioCreateResponseDTO {
    private String _id;
    private String message;
}
```

Feito isso, vamos literalmente extrair todo o corpo da resposta para esse objeto criado utilizando o recurso do RestAssured .extract().as()

```java
@Test
public void testCadastroDeUsuarioComDadosValidos() {

    UsuarioCreateResponseDTO responseBody = given()
        .baseUri("http://localhost:3000")
        .contentType(ContentType.JSON)
        .body(UsuarioFactory.usuarioComDadosValidos())
    .when()
        .post("/usuario")
    .then()
          .contentType(ContentType.JSON)
          .statusCode(HttpStatus.SC_OK)
          .extract().as(UsuarioCreateResponseDTO.class);
          
    Assertions.assertNotNull(responseBody._id)
    Assertions.assertEquals(responseBody.message, "Cadastro realizado com sucesso")
}
```