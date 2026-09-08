# Kifeito

O **Kifeito** é uma aplicação web de gerenciamento pessoal de tarefas.

O projeto utiliza uma arquitetura distribuída baseada em **microsserviços**, com responsabilidades separadas entre interface, API de entrada, usuários, tarefas e notificações.

O usuário pode criar e gerenciar suas próprias tarefas, acompanhar seu progresso por meio de um dashboard e receber lembretes por e-mail relacionados às tarefas agendadas.

---
<a id="indice"></a>

## 📋 Índice

1. [📌 Sobre o projeto](#sobre-o-projeto)
2. [🎯 Objetivo](#objetivo)
3. [✨ Principais funcionalidades](#principais-funcionalidades)
4. [🏗️ Arquitetura](#arquitetura)
5. [🔄 Fluxo da aplicação](#fluxo-da-aplicacao)
6. [🧩 Microsserviços](#microsservicos)
7. [📡 Comunicação](#comunicacao)
8. [🛠️ Tecnologias](#tecnologias)
9. [📚 Repositórios](#repositorios)
10. [📖 Documentação](#documentacao)
11. [▶️ Execução](#execucao)
12. [📦 Escopo da versão 1](#escopo-da-versao-1)
13. [📦 Versão 2](#versao-1)
14. [🧭 Princípios arquiteturais](#principios-arquiteturais)
15. [📄 Licença](#licenca)

---

<a id="sobre-o-projeto"></a>
# 📌 Sobre o projeto

O Kifeito é um **gerenciador pessoal de tarefas**, no qual cada usuário possui e gerencia exclusivamente suas próprias tarefas.

O projeto é dividido em componentes independentes:

```text
Kifeito
│
├── kifeito-frontend
├── kifeito-bff
├── kifeito-user
├── kifeito-tasks
└── kifeito-notification
```

Cada componente possui uma responsabilidade específica e pode evoluir de forma independente.

⬆️ [Voltar ao índice](#-índice)

---

<a id="objetivo"></a>
# 🎯 Objetivo

O objetivo do Kifeito é fornecer uma aplicação simples para gerenciamento pessoal de tarefas, aplicando conceitos de engenharia de software e arquitetura distribuída.

O projeto busca demonstrar:

- Separação clara de responsabilidades;
- Baixo acoplamento entre componentes;
- Comunicação síncrona e assíncrona;
- Isolamento dos dados por domínio;
- Autenticação e autorização;
- Persistência de dados;
- Mensageria;
- Testes;
- Containerização;
- CI/CD;
- Evolução incremental da arquitetura.

⬆️ [Voltar ao índice](#-índice)

---

<a id="principais-funcionalidades"></a>
# ✨ Principais funcionalidades

## Autenticação

- Cadastro de usuário;
- Login;
- Autenticação utilizando JWT;
- Logout;
- Proteção de recursos autenticados.

## Usuário

- Visualização dos próprios dados;
- Atualização de nome;
- Atualização de e-mail;
- Exclusão da própria conta.

## Tarefas

- Criar tarefas;
- Listar tarefas;
- Visualizar detalhes;
- Editar tarefas;
- Concluir tarefas;
- Reabrir tarefas;
- Cancelar tarefas;
- Reativar tarefas;
- Excluir tarefas.

## Dashboard

O usuário possui uma visão consolidada das suas tarefas, incluindo:

- Total de tarefas;
- Tarefas pendentes;
- Tarefas concluídas;
- Tarefas canceladas;
- Próximas tarefas;
- Tarefas atrasadas.

## Notificações

Tarefas com data de agendamento podem gerar lembretes.

Na versão 1, o lembrete é enviado por e-mail **uma hora antes do horário programado da tarefa**.

O processamento das notificações ocorre de forma assíncrona utilizando RabbitMQ.

⬆️ [Voltar ao índice](#-índice)

---

<a id="arquitetura"></a>
# 🏗️ Arquitetura

O Kifeito utiliza uma arquitetura baseada em microsserviços.

```text
                         ┌─────────────────────┐
                         │       Usuário       │
                         └──────────┬──────────┘
                                    │
                                    ▼
                         ┌─────────────────────┐
                         │      Frontend       │
                         │       Angular       │
                         └──────────┬──────────┘
                                    │
                                    │ HTTP
                                    ▼
                         ┌─────────────────────┐
                         │         BFF         │
                         └──────┬────────┬─────┘
                                │        │
                         HTTP   │        │   HTTP
                                ▼        ▼
                         ┌──────────┐ ┌──────────┐
                         │   User   │ │  Tasks   │
                         └──────────┘ └────┬─────┘
                                           │
                                           │ Events
                                           ▼
                                     ┌───────────┐
                                     │ RabbitMQ  │
                                     └─────┬─────┘
                                           │
                                           ▼
                                   ┌──────────────┐
                                   │ Notification │
                                   └──────┬───────┘
                                          │
                                          ▼
                                        E-mail
```

O frontend acessa somente o BFF. Os serviços internos não são expostos diretamente ao cliente.

⬆️ [Voltar ao índice](#-índice)

---

<a id="fluxo-da-aplicacao"></a>
# 🔄 Fluxo da aplicação

## Requisições do usuário

```text
Frontend
   ↓ HTTP
BFF
   ↓ HTTP
User / Tasks
```

O BFF funciona como ponto de entrada da aplicação para o frontend, adaptando o contrato público e encaminhando as operações aos serviços responsáveis.

## Autenticação

```text
Frontend
   ↓
BFF
   ↓
User
   ↓
JWT
   ↓
Frontend
```

Nas operações autenticadas, o token é enviado pelo frontend ao BFF.

## Criação de tarefa

```text
Frontend
   ↓
BFF
   ↓
Tasks
   ↓
PostgreSQL
```

Quando a tarefa possui `scheduledAt`, o Tasks pode publicar um evento para o RabbitMQ:

```text
Tasks
   ↓
RabbitMQ
   ↓
Notification
   ↓
PostgreSQL
   ↓
E-mail
```

O serviço Notification mantém o estado do lembrete e realiza o envio no momento apropriado.

⬆️ [Voltar ao índice](#-índice)

---

<a id="microsservicos"></a>
# 🧩 Microsserviços

## Frontend

Responsável pela interface web da aplicação.

Principais tecnologias:

- Angular;
- TypeScript;
- HTML;
- CSS;
- Angular Router;
- Angular HttpClient;
- Reactive Forms;
- RxJS.

O frontend consome exclusivamente a API do BFF.

**Repositório:** `kifeito-frontend`

---

## BFF

**Backend for Frontend** responsável por disponibilizar a API utilizada pelo frontend.

Responsabilidades:

- Receber requisições do frontend;
- Encaminhar chamadas para os serviços internos;
- Adaptar contratos;
- Centralizar a comunicação do frontend com o backend;
- Agregar informações para o Dashboard.

O BFF não possui banco de dados próprio e não implementa as regras de negócio dos domínios.

**Repositório:** `kifeito-bff`

---

## User

Serviço responsável pelo domínio de usuários e autenticação.

Responsabilidades:

- Cadastro;
- Autenticação;
- Consulta dos dados do usuário;
- Atualização da própria conta;
- Exclusão da conta;
- Emissão e validação relacionada à autenticação.

Principais tecnologias:

- Java;
- Spring Boot;
- Spring Security;
- JWT;
- PostgreSQL.

**Repositório:** `kifeito-user`

---

## Tasks

Serviço responsável pelo domínio das tarefas.

Responsabilidades:

- Criação de tarefas;
- Consulta;
- Atualização;
- Controle do ciclo de vida;
- Persistência;
- Publicação dos eventos relacionados às tarefas.

Estados da tarefa na versão 1:

```text
PENDING
COMPLETED
CANCELLED
```

Principais tecnologias:

- Java;
- Spring Boot;
- PostgreSQL;
- RabbitMQ.

**Repositório:** `kifeito-tasks`

---

## Notification

Serviço responsável pelo domínio de notificações e lembretes.

Responsabilidades:

- Receber eventos de tarefas;
- Criar lembretes;
- Reagendar lembretes;
- Cancelar lembretes;
- Controlar o estado dos lembretes;
- Enviar e-mails.

Estados dos lembretes:

```text
PENDING
SENT
CANCELLED
```

O serviço possui seu próprio banco de dados porque o estado do lembrete pertence ao domínio de Notification.

Principais tecnologias:

- Java;
- Spring Boot;
- PostgreSQL;
- RabbitMQ;
- Spring Mail;
- Thymeleaf;
- Spring Scheduling.

**Repositório:** `kifeito-notification`

⬆️ [Voltar ao índice](#-índice)

---

<a id="comunicacao"></a>
# 📡 Comunicação

O Kifeito utiliza dois modelos principais de comunicação.

## 📡 Comunicação síncrona

Utilizada nas operações que precisam de resposta imediata.

```text
Frontend
   ↓ HTTP
BFF
   ↓ HTTP
User / Tasks
```

Exemplos:

- Login;
- Cadastro;
- Consulta de usuário;
- Criação de tarefa;
- Consulta de tarefas;
- Atualização de tarefa.

## 📡 Comunicação assíncrona

Utilizada para eventos relacionados às notificações.

```text
Tasks
   ↓
RabbitMQ
   ↓
Notification
```

Eventos da V1:

```text
TaskCreated
TaskScheduledDateChanged
TaskCompleted
TaskReopened
TaskCancelled
TaskReactivated
TaskDeleted
```

O serviço Notification reage aos eventos e mantém apenas o estado pertencente ao seu próprio domínio.

⬆️ [Voltar ao índice](#-índice)

---

<a id="tecnologias"></a>
# 🛠️ Tecnologias

| Tecnologia | Utilização |
|---|---|
| Java | Backend |
| Spring Boot | Desenvolvimento dos microsserviços |
| Spring Security | Segurança e autenticação |
| JWT | Autenticação |
| Angular | Frontend |
| TypeScript | Desenvolvimento frontend |
| PostgreSQL | Persistência dos dados |
| RabbitMQ | Comunicação assíncrona |
| Spring Data JPA | Persistência nos serviços Java |
| Spring Cloud OpenFeign | Comunicação HTTP entre serviços |
| Spring Mail | Envio de e-mails |
| Thymeleaf | Templates de e-mail |
| Spring Scheduling | Agendamento dos lembretes |
| Docker | Containerização |
| Docker Compose | Execução conjunta dos componentes |
| GitHub Actions | CI |

⬆️ [Voltar ao índice](#-índice)

---

<a id="repositorios"></a>
# 📚 Repositórios

O projeto é dividido em repositórios independentes.

| Repositório | Responsabilidade |
|---|---|
| `kifeito-frontend` | Interface web |
| `kifeito-bff` | API de entrada para o frontend |
| `kifeito-user` | Domínio de usuários e autenticação |
| `kifeito-tasks` | Domínio de tarefas |
| `kifeito-notification` | Domínio de notificações e lembretes |

> Substitua `<seu-usuario>` pelos dados da sua conta do GitHub nos links abaixo.

### Frontend

[Kifeito Frontend](https://github.com/Jeffergs/kifeito-frontend)

Documentação: `kifeito-frontend/README.md`

### BFF

[Kifeito BFF](https://github.com/Jeffergs/kifeito-bff)

Documentação: `kifeito-bff/README.md`

### User

[Kifeito User](https://github.com/Jeffergs/kifeito-user)

Documentação: `kifeito-user/README.md`

### Tasks

[Kifeito Tasks](https://github.com/Jeffergs/kifeito-tasks)

Documentação: `kifeito-tasks/README.md`

### Notification

[Kifeito Notification](https://github.com/Jeffergs/kifeito-notification)

Documentação: `kifeito-notification/README.md`

⬆️ [Voltar ao índice](#-índice)

---

<a id="documentacao"></a>
# 📖 Documentação

A documentação técnica está distribuída entre os repositórios do projeto.

Cada microsserviço possui seu próprio README com informações sobre:

- Responsabilidade;
- Funcionalidades;
- Endpoints;
- Regras de negócio;
- Tecnologias;
- Estrutura;
- Configuração;
- Docker;
- Testes;
- Escopo.

Para compreender uma parte específica do sistema, consulte o README do respectivo repositório.

⬆️ [Voltar ao índice](#-índice)

---

<a id="execucao"></a>
# ▶️ Execução

O projeto foi estruturado para permitir a execução dos componentes em containers.

A infraestrutura inicial pode ser representada por:

```text
Docker Compose
│
├── Frontend
├── BFF
├── User
├── Tasks
├── Notification
├── PostgreSQL
└── RabbitMQ
```

Os serviços podem ser executados em uma única infraestrutura, mantendo sua separação lógica.

Essa abordagem permite desenvolver e demonstrar uma arquitetura de microsserviços sem introduzir infraestrutura desnecessariamente complexa.

⬆️ [Voltar ao índice](#-índice)

---
<a id="escopo-da-versao-1"></a>
# 📦 Escopo da versão 1

## Usuários

- Cadastro;
- Login;
- JWT;
- Consulta da própria conta;
- Atualização de nome;
- Atualização de e-mail;
- Exclusão da conta.

## Tarefas

- Criação;
- Consulta;
- Listagem;
- Atualização;
- Conclusão;
- Reabertura;
- Cancelamento;
- Reativação;
- Exclusão definitiva.

## Notificações

- Lembrete baseado em `scheduledAt`;
- Envio uma hora antes da tarefa;
- Reagendamento;
- Cancelamento;
- Controle de estado do lembrete;
- Envio por e-mail.

## Interface

- Cadastro;
- Login;
- Dashboard;
- Minhas tarefas;
- Detalhes da tarefa;
- Criação;
- Edição;
- Perfil;
- Estados de loading, sucesso e erro;
- Componentes reutilizáveis.

⬆️ [Voltar ao índice](#-índice)

---
<a id="versao-2"></a>
# 🚀 Versão 2

Possíveis evoluções:

- Tarefas recorrentes;
- Prioridades;
- Categorias;
- Tags;
- Filtros avançados;
- Busca;
- Paginação;
- Múltiplos lembretes;
- Recuperação de senha;
- Alteração de senha;
- Refresh Token;
- 2FA;
- OAuth2;
- Login social;
- PWA;
- Aplicação mobile.

⬆️ [Voltar ao índice](#-índice)

---
<a id="principios-arquiteturais"></a>
# 🧭 Princípios arquiteturais

O desenvolvimento do Kifeito segue princípios para orientar as decisões técnicas.

## Complexidade proporcional ao benefício

> **Toda decisão arquitetural deve trazer um benefício proporcional à complexidade que adiciona.**

Uma tecnologia ou padrão não deve ser introduzido apenas porque é utilizado em arquiteturas maiores.

A adoção deve estar relacionada a uma necessidade concreta do sistema.

## Single Source of Truth

> **Cada informação de negócio deve possuir uma única fonte dentro do domínio.**

Exemplo: `scheduledAt` pertence ao domínio de Tasks.

Notification não mantém uma cópia de `scheduledAt`. Notification mantém `scheduledFor`, que representa o momento em que o lembrete deve ser enviado.

## Domínio antes da implementação

> **O modelo de domínio deve representar a realidade do negócio, e não a conveniência da implementação.**

As entidades, responsabilidades e regras devem ser definidas a partir do comportamento esperado do negócio.

## Separação de responsabilidades

Cada serviço deve possuir uma responsabilidade clara:

```text
User
   → Usuários e autenticação

Tasks
   → Tarefas

Notification
   → Lembretes e notificações

BFF
   → Contrato de entrada para o frontend

Frontend
   → Interface e interação
```

## Evolução incremental

A arquitetura deve permitir evolução gradual.

A infraestrutura inicial não precisa possuir todos os componentes normalmente encontrados em sistemas distribuídos de grande escala.

Novas tecnologias devem ser introduzidas somente quando existir uma necessidade que justifique sua complexidade.

⬆️ [Voltar ao índice](#-índice)

---

<a id="licenca"></a>
# 📄 Licença

O Kifeito está sendo desenvolvido inicialmente para uso próprio e para um grupo limitado de usuários.

Apesar do uso inicial restrito, o projeto está sendo desenvolvido com arquitetura, práticas e estrutura voltadas para um produto comercial, podendo futuramente ser disponibilizado de forma mais ampla.

O código-fonte, a aplicação, a identidade visual, a documentação e demais componentes do projeto são de propriedade do próprio autor.

A utilização, cópia, modificação, distribuição ou comercialização de qualquer parte do projeto depende de autorização expressa do detentor dos direitos.

Consulte o arquivo `LICENSE` para os termos completos.

⬆️ [Voltar ao índice](#-índice)
