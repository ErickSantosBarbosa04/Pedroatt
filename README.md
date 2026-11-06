# API de Agendamento Odontológico

API REST em Java 21 e Spring Boot para cadastro e autenticação de dentistas, preparada para usar MySQL no Aiven e para build e deploy em Docker.

## O que foi preparado

- Configuração do banco e do segredo JWT por variáveis de ambiente, sem valores de acesso versionados.
- Endpoint público de saúde em `/actuator/health`, sem exibir detalhes internos.
- Teste de inicialização com H2 em memória, sem depender do banco remoto.
- `Dockerfile` multiestágio com Maven e Java 21.
- Workflow do GitHub Actions para executar `clean verify` e validar o build Docker em cada envio à branch `main`.
- `.gitignore` e `.dockerignore` para excluir arquivos locais, segredos e artefatos compilados.
- Limpeza dos arquivos compilados que estavam versionados.

## Tecnologias

- Java 21
- Spring Boot
- Spring Data JPA
- MySQL e Flyway
- Spring Security e JWT
- Actuator
- Maven Wrapper
- Docker
- GitHub Actions

## Estrutura do repositório

```text
.
├── .github/workflows/agendamento-odontologico-cicd.yml
├── api-odonto/
│   ├── .mvn/wrapper/maven-wrapper.properties
│   ├── Dockerfile
│   ├── mvnw / mvnw.cmd
│   ├── pom.xml
│   └── src/
│       ├── main/resources/db/migration/
│       └── test/resources/application.properties
├── .gitignore
└── README.md
```

## Executar localmente

Instale o JDK 21. O Maven Wrapper do projeto baixa e usa a versão necessária do Maven.

No PowerShell, a partir da pasta `api-odonto`, configure os dados do seu banco Aiven e inicie a aplicação:

```powershell
$env:DB_URL = "jdbc:mysql://HOST:PORT/defaultdb?sslMode=REQUIRED"
$env:DB_USERNAME = "USUARIO_DO_BANCO"
$env:DB_PASSWORD = "SENHA_DO_BANCO"
$env:JWT_SECRET = "SEGREDO_LONGO_E_ALEATORIO"
.\mvnw.cmd spring-boot:run
```

Substitua os valores de exemplo pelos dados exibidos no seu serviço MySQL. Não grave credenciais no código, no README, em commits ou em capturas de tela.

Para executar a verificação Maven:

```powershell
.\mvnw.cmd --batch-mode clean verify
```

Os testes usam um banco H2 temporário. Para executar a aplicação real, são necessários os dados do MySQL e um segredo JWT.

## Variáveis de ambiente

| Nome | Uso |
| --- | --- |
| `DB_URL` | URL JDBC do banco MySQL Aiven, com host, porta, banco `defaultdb` e SSL habilitado. |
| `DB_USERNAME` | Usuário do banco MySQL. |
| `DB_PASSWORD` | Senha do banco MySQL. |
| `JWT_SECRET` | Segredo privado usado para assinar tokens. Use um valor longo e aleatório. |
| `PORT` | Porta HTTP; opcional. Usa `8080` por padrão e aceita a porta definida pelo Render. |

## Docker

Na pasta `api-odonto`, construa a imagem:

```powershell
docker build -t agendamento-odontologico .
```

Para iniciar localmente, defina as variáveis de ambiente acima no terminal e execute:

```powershell
docker run --rm -p 8080:8080 -e DB_URL -e DB_USERNAME -e DB_PASSWORD -e JWT_SECRET -e PORT=8080 agendamento-odontologico
```

Com a aplicação no ar, verifique `http://localhost:8080/actuator/health`. A resposta esperada é `{"status":"UP"}` 


## Referências oficiais

- [Conectar Java ao Aiven for MySQL](https://aiven.io/docs/products/mysql/howto/connect-with-java)
- [Docker no Render](https://render.com/docs/docker)
- [Health checks do Render](https://render.com/docs/health-checks)
- [Variáveis de ambiente no Render](https://render.com/docs/configure-environment-variables)
- [Usar secrets no GitHub Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)

