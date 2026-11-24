# 🏥 Voll.med API — Segurança, Autenticação JWT e Controle de Acesso

API RESTful desenvolvida em **Java** com **Spring Boot** para gerenciamento de médicos, pacientes e usuários da clínica **Voll.med**, implementando autenticação *stateless* via **Spring Security** e geração/validação de tokens **JSON Web Token (JWT)**.

Projeto desenvolvido na disciplina **Tecnologias para Back-End** (Prof. Junio Figueirêdo).

---

## 🚀 Tecnologias Utilizadas

* **Java 21**
* **Spring Boot 3**
* **Spring Web** (API RESTful)
* **Spring Data JPA** (persistência e repositórios)
* **Spring Security** (autenticação, criptografia BCrypt e filtro de autorização)
* **Spring Validation** (`@Valid` e validação de DTOs)
* **Auth0 `java-jwt`** (geração e validação de tokens JWT - RFC 7519)
* **Flyway Migration** (versionamento e criação automática das tabelas no banco)
* **MySQL** (banco de dados relacional)
* **Lombok** (redução de código *boilerplate*)
* **SpringDoc OpenAPI / Swagger** (documentação interativa da API)

---

## 📁 Estrutura do Projeto

```text
projeto_p4/
├── src/
│   ├── main/
│   │   ├── java/med/voll/api/
│   │   │   ├── ApiApplication.java
│   │   │   ├── controller/
│   │   │   │   ├── AutenticacaoController.java    # Endpoint de login (/login)
│   │   │   │   ├── HelloController.java           # Endpoint de verificação
│   │   │   │   ├── MedicoController.java          # CRUD de médicos
│   │   │   │   ├── PacienteController.java        # CRUD de pacientes
│   │   │   │   └── UsuarioController.java         # Cadastro/gestão de usuários
│   │   │   ├── domain/
│   │   │   │   ├── endereco/                      # Entidade embutida e DTO de endereço
│   │   │   │   ├── medico/                        # Entidade, Repository, Enum e DTOs de médico
│   │   │   │   ├── paciente/                      # Entidade, Repository e DTOs de paciente
│   │   │   │   └── usuario/                       # Entidade (UserDetails), AutenticacaoService, Repository e DTOs
│   │   │   └── infra/
│   │   │       ├── exception/
│   │   │       │   └── TratadorDeErros.java       # Tratamento global de exceções HTTP
│   │   │       ├── security/
│   │   │       │   ├── DadosTokenJWT.java         # DTO de retorno do token
│   │   │       │   ├── SecurityConfigurations.java# Regras de acesso, BCrypt e ordem de filtros
│   │   │       │   ├── SecurityFilter.java        # Interceptação de requisições e validação do Bearer Token
│   │   │       │   └── TokenService.java          # Geração e validação de tokens JWT (HMAC256)
│   │   │       └── springdoc/
│   │   │           └── SpringDocConfigurations.java # Configuração de documentação OpenAPI/Swagger
│   │   └── resources/
│   │       ├── application.properties             # Configurações de desenvolvimento
│   │       ├── application-prod.properties        # Configurações de produção
│   │       └── db/migration/
│   │           ├── V1__create-table-medicos.sql
│   │           ├── V2__alter-table-medicos-add-column-telefone.sql
│   │           ├── V3__alter-table-medicos-add-column-ativo.sql
│   │           ├── V4__create-table-pacientes.sql
│   │           └── V5__create-table-usuarios.sql
└── pom.xml
```

---

## 🔐 Fluxo de Autenticação e Controle de Acesso

1. **Login (`POST /login`):**
   * O cliente envia `login` e `senha` no corpo da requisição (`DadosAutenticacao`).
   * O `AutenticacaoController` cria um `UsernamePasswordAuthenticationToken` e aciona o `AuthenticationManager`.
   * O Spring Security utiliza o `AutenticacaoService` (`UserDetailsService`) para buscar o usuário no banco e validar a senha criptografada com `BCryptPasswordEncoder`.
   * Estando válido, o `TokenService` gera um **Token JWT** assinado com `HMAC256`, contendo o emissor (`API Voll.med`), o *subject* (login do usuário) e expiração de 2 horas, retornando `200 OK` com `{"token": "..."}`.

2. **Interceptação e Autorização (`SecurityFilter` + `SecurityConfigurations`):**
   * A aplicação opera no modo **`STATELESS`** (sem sessão no servidor).
   * Apenas rotas públicas liberadas (como `POST /login`) podem ser acessadas sem autenticação.
   * Nas demais requisições, o `SecurityFilter` (executado antes do `UsernamePasswordAuthenticationFilter`) extrai o token do cabeçalho `Authorization: Bearer <token>`, valida sua assinatura e expiração via `TokenService.getSubject(...)`, carrega o usuário e autentica a requisição no `SecurityContextHolder`.

---

## ⚙️ Como Executar o Projeto Localmente

### 1. Pré-requisitos
* **JDK 21** instalado
* **Maven** (ou uso do wrapper `./mvnw` / IDE como IntelliJ IDEA)
* **MySQL Server** em execução

### 2. Configurar o Banco de Dados
Crie o banco de dados no MySQL:
```sql
CREATE DATABASE IF NOT EXISTS p4_back_2025;
```

### 3. Configurar as Propriedades (`application.properties`)
No arquivo `src/main/resources/application.properties`, ajuste o usuário, senha do MySQL e a chave secreta do JWT conforme o seu ambiente:
```properties
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/p4_back_2025
spring.datasource.username=root
spring.datasource.password=SUA_SENHA_MYSQL

api.security.token.secret=${JWT_SECRET:12345678}
```
> **Nota:** Ao iniciar a aplicação pela primeira vez, o **Flyway** executará automaticamente todas as migrations (`V1` a `V5`) e criará as tabelas no banco.

### 4. Iniciar a Aplicação
Execute a classe principal `ApiApplication.java` pela sua IDE ou via terminal:
```bash
./mvnw spring-boot:run
```
A API ficará disponível em `http://localhost:8080`.

---

## 🧪 Como Testar os Endpoints (Insomnia / Postman)

1. **Obter o Token JWT:**
   * **Método:** `POST`
   * **URL:** `http://localhost:8080/login`
   * **Body (JSON):**
     ```json
     {
       "login": "ana.souza@voll.med",
       "senha": "123456"
     }
     ```
   * **Resposta esperada (`200 OK`):**
     ```json
     {
       "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
     }
     ```

2. **Acessar Rotas Protegidas (ex.: Listar Médicos):**
   * **Método:** `GET`
   * **URL:** `http://localhost:8080/medicos`
   * **Header / Auth:** Configure **Bearer Token** e informe o token obtido no passo anterior (`Authorization: Bearer <seu_token>`).
   * Sem o token (ou com token inválido/expirado), a API bloqueia o acesso.

---

## 📌 Principais Endpoints

| Recurso | Método | Endpoint | Acesso |
| :--- | :--- | :--- | :--- |
| Autenticação | `POST` | `/login` | Público |
| Usuários | `POST` | `/usuarios` | Conforme regra de segurança |
| Médicos | `GET`, `POST`, `PUT`, `DELETE` | `/medicos` | Autenticado (Bearer JWT) |
| Pacientes | `GET`, `POST`, `PUT`, `DELETE` | `/pacientes` | Autenticado (Bearer JWT) |
| Documentação | `GET` | `/swagger-ui.html` | Conforme regra de segurança |
