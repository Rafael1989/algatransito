# algatransito-api

API REST desenvolvida com Spring Boot para gerenciamento de proprietários, veículos, apreensões e autuações.

## Visão geral

O projeto expõe endpoints para:

- cadastrar, consultar, atualizar e remover proprietários;
- cadastrar e consultar veículos vinculados a um proprietário;
- apreender e remover apreensão de veículos;
- registrar e listar autuações de um veículo.

A aplicação usa MySQL como banco de dados, Flyway para versionamento de schema e Spring Data JPA para persistência.

## Stack utilizada

- Java 17
- Spring Boot 3.1.0
- Spring Web
- Spring Data JPA
- Bean Validation
- Flyway
- MySQL
- ModelMapper
- Lombok
- Maven Wrapper (`mvnw` / `mvnw.cmd`)

## Requisitos

- JDK 17
- MySQL em execução
- Maven não é obrigatório, pois o projeto já inclui Maven Wrapper

## Configuração atual

O arquivo `src/main/resources/application.properties` está configurado assim:

```properties
spring.application.name=algatrasito-api
spring.datasource.url=jdbc:mysql://localhost/algatransito?createDatabaseIfNotExist=true&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=root
```

### Observações

- o banco `algatransito` é criado automaticamente se não existir;
- as migrations do Flyway são aplicadas na inicialização;
- se seu MySQL não usa `root/root`, ajuste as credenciais antes de subir a aplicação.

## Como executar

No Windows:

```powershell
.\mvnw.cmd spring-boot:run
```

Em Linux/macOS:

```bash
./mvnw spring-boot:run
```

A aplicação sobe, por padrão, em:

```text
http://localhost:8080
```

## Como rodar os testes

No Windows:

```powershell
.\mvnw.cmd test
```

Em Linux/macOS:

```bash
./mvnw test
```

Atualmente o projeto possui apenas um teste de contexto em `src/test/java/com/algaworks/algatransito/AlgatrasitoApiApplicationTests.java`.

## Estrutura principal do projeto

```text
src/main/java/com/algaworks/algatransito
├── api
│   ├── assembler
│   ├── controller
│   ├── exceptionhandler
│   └── model
├── common
├── domain
│   ├── exception
│   ├── model
│   ├── repository
│   └── service
└── AlgatrasitoApiApplication.java
```

## Modelo de domínio

### Proprietário

Campos da entidade `Proprietario`:

- `id`
- `nome`
- `email`
- `telefone`

Regras importantes:

- `nome` é obrigatório e aceita até 60 caracteres;
- `email` é obrigatório, deve ser válido e é único;
- `telefone` é obrigatório e aceita até 20 caracteres;
- no banco, a coluna foi renomeada para `fone` pela migration `V002`.

### Veículo

Campos da entidade `Veiculo`:

- `id`
- `proprietario`
- `marca`
- `modelo`
- `placa`
- `status`
- `dataCadastro`
- `dataApreensao`
- `autuacoes`

Regras importantes:

- todo veículo pertence a um proprietário existente;
- a placa deve ser única;
- o status inicial ao cadastrar é `REGULAR`;
- `dataCadastro` é preenchida automaticamente;
- ao apreender, o status muda para `APREENDIDO` e `dataApreensao` recebe a data/hora atual;
- ao remover a apreensão, o status volta para `REGULAR` e `dataApreensao` vira `null`.

### Autuação

Campos da entidade `Autuacao`:

- `id`
- `veiculo`
- `descricao`
- `valorMulta`
- `dataOcorrencia`

Regras importantes:

- toda autuação pertence a um veículo existente;
- `dataOcorrencia` é preenchida automaticamente ao registrar a autuação.

### Status de veículo

Valores possíveis do enum `StatusVeiculo`:

- `REGULAR`
- `APREENDIDO`

## Banco de dados e migrations

As migrations ficam em `src/main/resources/db/migration`:

- `V001__cria-tabela-proprietario.sql`
- `V002__renomeia-coluna-telefone.sql`
- `V003__cria-tabela-veiculo.sql`
- `V004__cria-tabela-autuacao.sql`

### Tabelas criadas

#### `proprietario`

- `id`
- `nome`
- `email` (único)
- `fone`

#### `veiculo`

- `id`
- `proprietario_id` (FK para `proprietario.id`)
- `marca`
- `modelo`
- `placa` (único)
- `status`
- `data_cadastro`
- `data_apreensao`

#### `autuacao`

- `id`
- `veiculo_id` (FK para `veiculo.id`)
- `descricao`
- `valor_multa`
- `data_ocorrencia`

## Endpoints

## Proprietários

### Listar proprietários

```http
GET /proprietarios
```

Retorna uma lista de `Proprietario`.

### Buscar proprietário por ID

```http
GET /proprietarios/{proprietarioId}
```

- retorna `200 OK` quando encontrado;
- retorna `404 Not Found` quando não encontrado.

### Cadastrar proprietário

```http
POST /proprietarios
Content-Type: application/json
```

Exemplo de payload:

```json
{
  "nome": "Carlos Almeida",
  "email": "carlos.almeida@example.com",
  "telefone": "11987654321"
}
```

Retorna `201 Created`.

### Atualizar proprietário

```http
PUT /proprietarios/{proprietarioId}
Content-Type: application/json
```

Exemplo de payload:

```json
{
  "nome": "Carlos Almeida",
  "email": "carlos.almeida@example.com",
  "telefone": "11987654321"
}
```

- retorna `200 OK` quando atualizado;
- retorna `404 Not Found` quando o ID não existe.

### Remover proprietário

```http
DELETE /proprietarios/{proprietarioId}
```

- retorna `204 No Content` quando removido;
- retorna `404 Not Found` quando o ID não existe;
- pode retornar `409 Conflict` se o recurso estiver em uso.

## Veículos

### Listar veículos

```http
GET /veiculos
```

Retorna uma lista de `VeiculoModel`.

Exemplo de resposta:

```json
[
  {
    "id": 1,
    "proprietario": {
      "id": 1,
      "nome": "Carlos Almeida"
    },
    "marca": "Honda",
    "modelo": "Civic",
    "numeroPlaca": "ABC1D23",
    "status": "REGULAR",
    "dataCadastro": "2026-03-25T10:00:00Z",
    "dataApreensao": null
  }
]
```

> Observação: no input o campo é `placa`, mas no output o campo exposto é `numeroPlaca`.

### Buscar veículo por ID

```http
GET /veiculos/{veiculoId}
```

- retorna `200 OK` quando encontrado;
- retorna `404 Not Found` quando não encontrado.

### Cadastrar veículo

```http
POST /veiculos
Content-Type: application/json
```

Exemplo de payload:

```json
{
  "marca": "Honda",
  "modelo": "Civic",
  "placa": "ABC1D23",
  "proprietario": {
    "id": 1
  }
}
```

Regras de validação observadas no DTO `VeiculoInput`:

- `marca`: obrigatória, máximo de 20 caracteres;
- `modelo`: obrigatório, máximo de 20 caracteres;
- `placa`: obrigatória e deve seguir o padrão `[A-Z]{3}[0-9][0-9A-Z][0-9]{2}`;
- `proprietario.id`: obrigatório.

Retorna `201 Created` com o `VeiculoModel` persistido.

### Apreender veículo

```http
PUT /veiculos/{veiculoId}/apreensao
```

Retorna `204 No Content`.

Regra de negócio:

- se o veículo já estiver apreendido, a API retorna erro de negócio (`400 Bad Request`).

### Remover apreensão do veículo

```http
DELETE /veiculos/{veiculoId}/apreensao
```

Retorna `204 No Content`.

Regra de negócio:

- se o veículo não estiver apreendido, a API retorna erro de negócio (`400 Bad Request`).

## Autuações

### Registrar autuação de um veículo

```http
POST /veiculos/{veiculoId}/autuacoes
Content-Type: application/json
```

Exemplo de payload:

```json
{
  "descricao": "Estacionamento em local proibido",
  "valorMulta": 195.23
}
```

Regras de validação observadas no DTO `AutuacaoInput`:

- `descricao`: obrigatória;
- `valorMulta`: obrigatório e positivo.

Retorna `201 Created` com a autuação registrada.

### Listar autuações de um veículo

```http
GET /veiculos/{veiculoId}/autuacoes
```

Retorna uma lista de `AutuacaoModel` do veículo informado.

## Tratamento de erros

A API centraliza erros em `ApiExceptionHandler` usando `ProblemDetail`.

### Erros de validação

Quando há campos inválidos, a API responde com `400 Bad Request` e estrutura semelhante a:

```json
{
  "type": "https://algatransito.com/erros/campos-invalidos",
  "title": "Um ou mais campos estão inválidos",
  "status": 400,
  "fields": {
    "placa": "deve ser uma placa válida"
  }
}
```

Mensagens configuradas em `src/main/resources/messages.properties`:

- `NotBlank=é obrigatório`
- `NotNull=é obrigatório`
- `Email=deve ser um e-mail válido`
- `Size=deve ter no mínimo {2} e no máximo {1} caracteres`
- `Pattern.placa=deve ser uma placa válida`

### Erros de negócio

São lançados como `NegocioException` e retornam `400 Bad Request`, por exemplo:

- proprietário não encontrado;
- veículo já se encontra apreendido;
- veículo não está apreendido;
- e-mail já cadastrado;
- placa já cadastrada;
- tentativa de cadastrar veículo com ID informado.

### Entidade não encontrada

Quando uma entidade procurada pelo serviço não existe, a API retorna `404 Not Found` com tipo:

```text
https://algatransito.com/erros/nao-encontrado
```

### Recurso em uso

Violações de integridade do banco retornam `409 Conflict` com tipo:

```text
https://algatransito.com/erros/recurso-em-uso
```

## Exemplos rápidos com curl

### Criar proprietário

```bash
curl -X POST http://localhost:8080/proprietarios \
  -H "Content-Type: application/json" \
  -d '{
    "nome": "Carlos Almeida",
    "email": "carlos.almeida@example.com",
    "telefone": "11987654321"
  }'
```

### Criar veículo para um proprietário existente

```bash
curl -X POST http://localhost:8080/veiculos \
  -H "Content-Type: application/json" \
  -d '{
    "marca": "Toyota",
    "modelo": "Corolla",
    "placa": "BRA2E19",
    "proprietario": {
      "id": 1
    }
  }'
```

### Registrar autuação

```bash
curl -X POST http://localhost:8080/veiculos/1/autuacoes \
  -H "Content-Type: application/json" \
  -d '{
    "descricao": "Excesso de velocidade",
    "valorMulta": 293.47
  }'
```

## Observações sobre o estado atual do código

- `ProprietarioController` expõe diretamente a entidade `Proprietario` nas operações de proprietário;
- `VeiculoController` e `AutuacaoController` usam DTOs e assemblers com `ModelMapper`;
- existe um mapeamento customizado de `Veiculo.placa` para `VeiculoModel.numeroPlaca` em `ModelMapperConfig`;
- o repositório de proprietários possui também o método `findByNomeContaining`, embora ele não esteja exposto por endpoint no momento;
- o arquivo `HELP.md` ainda contém o conteúdo padrão gerado pelo Spring Initializr.

## Próximos passos sugeridos

- adicionar testes de integração para os endpoints;
- documentar a API com OpenAPI/Swagger;
- externalizar credenciais do banco via variáveis de ambiente;
- criar perfis para desenvolvimento e teste.

