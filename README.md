# ERD — Entity Relationship Diagram Tool

Ferramenta web colaborativa para criação e gerenciamento de diagramas entidade-relacionamento (DER). Permite que equipes modelem schemas de banco de dados visualmente e em tempo real.

## Arquitetura

O projeto é composto por dois serviços principais orquestrados via Docker Compose:

| Serviço | Tecnologia | Porta |
|---|---|---|
| **erd-client** | Angular 14 + Nginx | 4200 |
| **erd-core** | Spring Boot 3.4 + Java 21 | 8080 |
| **PostgreSQL** | postgres:15-alpine | 5432 |
| **MongoDB** | mongo:6 | 27017 |

### Por que dois bancos?

- **PostgreSQL** — dados relacionais: usuários, projetos e membros de equipe.
- **MongoDB** — dados do diagrama (nós e arestas do GoJS), que variam estruturalmente e são salvos/lidos como documentos.

## Funcionalidades

- **Autenticação** — cadastro, login e logout com JWT armazenado em cookies HttpOnly; refresh token automático.
- **Projetos** — criação, edição e exclusão de projetos com controle de acesso por papel (OWNER / EDITOR / VIEWER).
- **Gerenciamento de equipe** — convite e remoção de membros, alteração de papel.
- **Editor de diagramas** — canvas interativo com GoJS para criar entidades, atributos e relacionamentos. Suporta os tipos de dados: `INTEGER`, `BIGINT`, `DECIMAL`, `VARCHAR`, `TEXT`, `BOOLEAN`, `DATE`, `TIMESTAMP`, `UUID`, entre outros.
- **Colaboração em tempo real** — via WebSocket (STOMP sobre SockJS): bloqueio de entidade durante edição, notificação de entrada/saída de usuários.
- **DDL import/export** — importa um script SQL e gera o diagrama; exporta o diagrama como DDL SQL.

## Pré-requisitos

- [Docker](https://www.docker.com/) e Docker Compose instalados.

## Executando com Docker (recomendado)

A partir da raiz deste repositório:

```bash
docker compose up --build
```

> O primeiro build pode levar alguns minutos enquanto o Maven baixa as dependências. As execuções seguintes são mais rápidas graças ao volume `maven_cache`.

Acesse a aplicação em **http://localhost:4200**.

### Comandos úteis

```bash
# Rodar em background
docker compose up -d

# Acompanhar logs de um serviço específico
docker compose logs -f erd-core

# Parar todos os serviços
docker compose down

# Parar e remover volumes (reseta os dados dos bancos)
docker compose down -v

# Rebuildar apenas um serviço
docker compose up --build erd-client
```

## Executando localmente (sem Docker)

### erd-core (backend)

**Pré-requisitos:** Java 21, Maven, PostgreSQL e MongoDB rodando localmente.

Configure as credenciais em `erd-core/src/main/resources/application.yml`, depois execute:

```bash
cd erd-core
./mvnw spring-boot:run
```

Ou suba apenas os bancos via Docker e aponte o backend para `localhost`:

```bash
cd erd-core
docker compose -f docker-compose-local.yml up -d
./mvnw spring-boot:run
```

### erd-client (frontend)

**Pré-requisitos:** Node.js, npm e Angular CLI instalados.

```bash
cd erd-client
npm install
ng serve
```

Acesse em **http://localhost:4200**. O cliente espera o backend em `http://localhost:8080`.

## Stack tecnológica

### Frontend (`erd-client`)

| Tecnologia | Versão | Uso |
|---|---|---|
| Angular | 14 | Framework principal |
| TypeScript | ~4.7 | Linguagem |
| GoJS | ^2.3 | Canvas interativo de diagramas |
| SockJS + StompJS | 1.6 / 7.1 | WebSocket para colaboração |
| Bootstrap | 4.6 | Estilização |

### Backend (`erd-core`)

| Tecnologia | Versão | Uso |
|---|---|---|
| Java | 21 | Linguagem |
| Spring Boot | 3.4.2 | Framework principal |
| Spring Security + JJWT | 0.12.6 | Autenticação JWT |
| Spring WebSocket (STOMP) | — | Colaboração em tempo real |
| Spring Data JPA | — | ORM para PostgreSQL |
| Spring Data MongoDB | — | Persistência dos diagramas |
| ModelMapper | 3.2.1 | Mapeamento DTO ↔ entidade |

## Endpoints da API

| Método | Rota | Descrição |
|---|---|---|
| `POST` | `/api/user` | Cadastro de usuário |
| `POST` | `/api/auth/login` | Login |
| `POST` | `/api/auth/logout` | Logout |
| `POST` | `/api/auth/refresh-token` | Renovar access token |
| `POST` | `/api/project` | Criar projeto |
| `GET` | `/api/project/user-email/{email}` | Listar projetos do usuário |
| `GET` | `/api/project/{id}` | Detalhes do projeto |
| `PUT` | `/api/project` | Atualizar projeto |
| `DELETE` | `/api/project/{id}` | Deletar projeto |
| `POST` | `/api/project/team-member` | Adicionar membro |
| `PUT` | `/api/project/team-member` | Atualizar papel do membro |
| `DELETE` | `/api/project/team-member/{memberId}/project/{projectId}` | Remover membro |
| `POST` | `/api/diagram` | Salvar diagrama |
| `GET` | `/api/diagram/{projectId}` | Carregar diagrama |
| `POST` | `/api/ddl/import` | Importar DDL SQL |
| `GET` | `/api/ddl/export/{projectId}` | Exportar DDL SQL |
| `WS` | `/ws` (STOMP `/app/send`) | Sincronização em tempo real |

## Repositórios

- Frontend: [erd-client](../erd-client)
- Backend: [erd-core](../erd-core)
