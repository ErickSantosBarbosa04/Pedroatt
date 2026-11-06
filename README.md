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

Com a aplicação no ar, verifique `http://localhost:8080/actuator/health`. A resposta esperada é `{"status":"UP"}` quando o banco estiver acessível.

## Integração contínua

O arquivo `.github/workflows/agendamento-odontologico-cicd.yml` executa no envio para `main` ou manualmente pelo GitHub Actions. Ele configura Java 21, roda `./mvnw --batch-mode clean verify` e constrói a imagem Docker. Os testes usam H2 em memória; o workflow não precisa receber senhas de produção.

## Publicação no Aiven e Render

1. Crie um serviço MySQL no Aiven e use o banco `defaultdb`.
2. Copie do Aiven o host, a porta, o usuário e a senha. Monte `DB_URL` com o endereço JDBC e TLS/SSL habilitado.
3. No Render, crie um **Web Service** conectado a este repositório e selecione **Docker** como ambiente.
4. Informe `api-odonto` como **Root Directory**, pois é nessa pasta que estão o `Dockerfile` e o `pom.xml`.
5. No painel **Environment** do serviço Render, cadastre `DB_URL`, `DB_USERNAME`, `DB_PASSWORD` e `JWT_SECRET`.
6. Configure o caminho de health check como `/actuator/health` e faça o deploy.
7. Confirme que o serviço está ativo e que `https://SEU-SERVICO.onrender.com/actuator/health` retorna `UP`.

O Render fornece a variável `PORT` para o serviço web. A aplicação escuta em `0.0.0.0` e usa essa porta automaticamente.

## Verificação e entrega

O fluxo previsto de validação é `mvnw clean verify`, build Docker e conferência do endpoint de saúde no serviço publicado. Conforme a validação informada pelo autor, os testes e verificações do projeto passaram.

Para entregar, compartilhe o link do repositório GitHub e, depois da publicação, o link público do serviço Render com `/actuator/health`. Não inclua valores dos segredos na entrega.

## Referências oficiais

- [Conectar Java ao Aiven for MySQL](https://aiven.io/docs/products/mysql/howto/connect-with-java)
- [Docker no Render](https://render.com/docs/docker)
- [Health checks do Render](https://render.com/docs/health-checks)
- [Variáveis de ambiente no Render](https://render.com/docs/configure-environment-variables)
- [Usar secrets no GitHub Actions](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets)

