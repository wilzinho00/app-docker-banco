# Mural de Mensagens — Docker, CI/CD e Azure

Aplicação web desenvolvida com Flask e MariaDB, utilizando Docker Compose para execução dos serviços e GitHub Actions para automação do processo de integração e entrega contínua (CI/CD).

O projeto demonstra a construção de uma aplicação com banco de dados, publicação de imagem Docker, execução de testes automatizados e atualização da aplicação na nuvem Microsoft Azure.

## Tecnologias utilizadas

- **Python / Flask:** desenvolvimento do backend e das rotas da aplicação.
- **HTML:** interface do mural de mensagens.
- **MariaDB:** armazenamento das mensagens.
- **Docker:** criação e execução dos containers.
- **Docker Compose:** gerenciamento da aplicação e do banco de dados.
- **Docker Hub:** armazenamento e distribuição da imagem Docker.
- **GitHub Actions:** automação da pipeline CI/CD.
- **Azure Container Apps:** hospedagem da aplicação na nuvem.

## Estrutura do projeto

```text
app-docker-banco/
│
├── .github/
│   └── workflows/
│       └── pipeline.yml
│
├── templates/
│   └── index.html
│
├── app.py
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env
├── .gitignore
└── README.md
```

## Executar localmente

### Pré-requisitos

- Docker Desktop instalado e em funcionamento.
- Docker Compose disponível.

### Configuração

Crie um arquivo `.env` na raiz do projeto com as variáveis necessárias:

```env
DB_PASSWORD=sua_senha
DB_ROOT_PASSWORD=sua_senha_root
```

Não compartilhe nem envie esse arquivo ao GitHub. Ele contém credenciais de acesso ao banco de dados.

### Inicialização

Na pasta do projeto, execute:

```bash
docker compose up --build -d
```

A aplicação ficará disponível em:

http://localhost:8081

Para verificar os containers:

```bash
docker compose ps
```

Para encerrar os serviços:

```bash
docker compose down
```

**Atenção:** o comando `docker compose down -v` também remove os volumes associados e pode apagar os dados armazenados no banco local.

## Banco de dados

A aplicação utiliza MariaDB para armazenar as mensagens enviadas pelos usuários.

No ambiente Docker Compose, o banco utiliza o volume `dados_mariadb`, permitindo manter os dados entre reinicializações dos containers, desde que o volume não seja removido.

## Pipeline CI/CD — GitHub Actions

O arquivo `.github/workflows/pipeline.yml` define a automação do projeto.

A pipeline é executada quando ocorre um push na branch `main` ou quando é iniciada manualmente pelo GitHub Actions.

### Etapas da pipeline

**1. Build e publicação**

- Valida a configuração do Docker Compose.
- Constrói a imagem da aplicação.
- Realiza autenticação no Docker Hub.
- Publica a imagem `wilzinho/mural-flask:latest`.

**2. Testes automatizados**

- Baixa a imagem publicada no Docker Hub.
- Inicializa a aplicação Flask e o banco MariaDB.
- Verifica se a aplicação responde corretamente por HTTP.
- Encerra os containers de teste, mesmo quando ocorre uma falha.

**3. Deploy no Azure**

- Realiza autenticação no Microsoft Azure.
- Atualiza a imagem da aplicação `mural-flask` no Azure Container Apps.
- Utiliza a imagem publicada no Docker Hub.

O deploy depende da conclusão bem-sucedida da etapa de testes.

## Configuração dos Secrets no GitHub

Para executar a pipeline, configure os seguintes secrets em:

**GitHub → Settings → Secrets and variables → Actions**

| Secret                  | Finalidade                                   |
| ----------------------- | -------------------------------------------- |
| `DOCKERHUB_TOKEN`       | Autenticação no Docker Hub.                  |
| `TEST_DB_PASSWORD`      | Senha do usuário do banco durante os testes. |
| `TEST_DB_ROOT_PASSWORD` | Senha root do banco durante os testes.       |
| `AZURE_CREDENTIALS`     | Credenciais de autenticação no Azure.        |

O job de deploy utiliza o ambiente `production` do GitHub Actions. Caso estejam configurados revisores obrigatórios nesse ambiente, a aprovação será necessária antes da execução do deploy.

## Implantação na nuvem — Microsoft Azure

A aplicação web está hospedada no Azure Container Apps.

**Aplicação:** `mural-flask`

**Grupo de recursos:** `rg-terraform-aula10`

**Ambiente:** `cae-mural-wilzinho`

**Imagem:** `wilzinho/mural-flask:latest`

**Endereço público:**

https://mural-flask.wonderfulgrass-e49d09d0.brazilsouth.azurecontainerapps.io/

O banco MariaDB está configurado em um Container App separado, chamado `mural-db`, com comunicação interna no ambiente do Azure.

A pipeline atualiza a aplicação Flask. O banco de dados é um recurso separado e não é recriado pelo job de deploy.

## Segurança

- Credenciais são armazenadas em variáveis de ambiente e Secrets do GitHub.
- O arquivo `.env` não deve ser versionado.
- A autenticação no Docker Hub utiliza um token.
- A autenticação no Azure utiliza credenciais armazenadas nos Secrets do GitHub.
- Senhas e tokens não devem ser publicados no código-fonte.

## Objetivo do projeto

Demonstrar, na prática, a integração entre desenvolvimento web, banco de dados, containers, automação CI/CD e implantação de aplicações em ambiente de nuvem.

O projeto reúne conceitos de desenvolvimento, infraestrutura e automação utilizados em processos modernos de entrega de software.

---

**Desenvolvido por Wilciano Nascimento (Wilzinho).**
