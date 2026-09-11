# DSVendas

![Java](https://img.shields.io/badge/Java-11-orange?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.7.18-6DB33F?logo=springboot&logoColor=white)
![React](https://img.shields.io/badge/React-17-61DAFB?logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-4.2-3178C6?logo=typescript&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-blue.svg)

Dashboard de análise de desempenho de vendas, com um back end REST em **Spring Boot** e um front end em **React + TypeScript**. O projeto foi originalmente desenvolvido durante o evento "Semana Spring React", promovido pela escola [DevSuperior](https://instagram.com/devsuperior.ig), como exercício de uma API REST consumida por uma SPA com gráficos e paginação.

## Descrição

A aplicação expõe relatórios de vendas de uma equipe comercial fictícia: valor total vendido por vendedor, taxa de sucesso (negócios fechados / clientes visitados) e uma listagem paginada de todas as vendas. O front end consome essa API e apresenta os dados em gráficos (barras e rosca) e em uma tabela paginada.

## Funcionalidades

Com base no código real do projeto:

- Listagem paginada e ordenada de vendas (`GET /sales`), com data, vendedor, clientes visitados, negócios fechados e valor.
- Soma do valor vendido agrupado por vendedor (`GET /sales/amount-by-seller`), usada no gráfico de rosca.
- Taxa de sucesso agrupada por vendedor (`GET /sales/success-by-seller`), usada no gráfico de barras.
- Listagem de vendedores (`GET /sellers`).
- Dashboard React com dois gráficos (ApexCharts) e uma tabela de vendas paginada.
- Tela inicial com apresentação do projeto e link para o dashboard.

> Todos os endpoints da API são somente leitura (`GET`) — não existe cadastro, edição ou exclusão de dados pela aplicação.

## Tecnologias utilizadas

**Back end**

- Java 11
- Spring Boot 2.7.18 (Web, Data JPA, Security)
- Hibernate / JPA
- PostgreSQL (produção/dev) e H2 (testes, em memória)
- Maven (com Maven Wrapper)

**Front end**

- React 17 + TypeScript
- React Router DOM 5
- Axios
- ApexCharts (via `react-apexcharts`)
- Bootstrap 4
- date-fns

## Pré-requisitos

- JDK 11 (o projeto compila e roda também em JDKs mais novos, como o 21, usando o Maven Wrapper)
- Node.js 16+ e npm
- PostgreSQL (apenas para rodar o back end com o perfil `dev`; os testes usam H2 em memória e não exigem banco externo)

## Como executar o projeto

### Back end

```bash
cd backend
cp .env.example .env        # edite com suas credenciais locais
export $(grep -v '^#' .env | xargs)
./mvnw spring-boot:run
```

A API sobe por padrão em `http://localhost:8080`. O perfil ativo é controlado pela variável `APP_PROFILE` (`test` por padrão — veja `application.properties`); para desenvolvimento local com PostgreSQL use `APP_PROFILE=dev`.

### Front end

```bash
cd frontend
cp .env.example .env        # ajuste REACT_APP_BACKEND_URL se necessário
npm install
npm start
```

A aplicação sobe por padrão em `http://localhost:3000` e consome a API em `http://localhost:8080` (configurável via `REACT_APP_BACKEND_URL`).

### Docker (back end + front end + PostgreSQL)

Também é possível subir a stack completa com Docker, sem instalar Java, Node ou PostgreSQL localmente:

```bash
docker compose up --build
```

Isso sobe três serviços:

- `postgres` — PostgreSQL 15, com o schema (`backend/create.sql`) e alguns dados de exemplo (`data.sql`) carregados automaticamente na primeira subida.
- `backend` — API Spring Boot (perfil `dev`, conectada ao `postgres`), disponível em `http://localhost:8080`.
- `frontend` — build de produção do React servido via Nginx em `http://localhost:3000`. O Nginx repassa as chamadas à API (`/sales`, `/sellers`) para o serviço `backend`, então não é preciso configurar `REACT_APP_BACKEND_URL`.

Também existe um workflow de CI (`.github/workflows/ci.yml`) que roda `mvn test` (perfil `test`, com H2 em memória) e `npm run build` a cada push/PR.

## Instalação

1. Clone o repositório.
2. Configure as variáveis de ambiente do back end (`backend/.env.example` → `backend/.env`) e do front end (`frontend/.env.example` → `frontend/.env`), preenchendo com valores locais.
3. Suba um banco PostgreSQL local (ou use Docker) e crie o schema — para uso local sem PostgreSQL, rode com `APP_PROFILE=test`, que usa H2 em memória com dados de exemplo já populados via `data.sql`.
4. Instale as dependências do front end com `npm install` dentro de `frontend/`.

## Como rodar os testes

**Back end**

```bash
cd backend
./mvnw test
```

Existe apenas um teste automatizado (`DsvendasApplicationTests`), que valida a subida do contexto do Spring. Não há testes unitários ou de integração cobrindo os serviços, controllers ou repositórios — ampliar essa cobertura é um item do roadmap abaixo.

**Front end**

```bash
cd frontend
npm test
```

O projeto não possui nenhum teste de front end configurado até o momento (nenhum arquivo `*.test.tsx` ou em `__tests__`).

## Estrutura de pastas

```
dashboard-vendas-spring-react/
├── backend/
│   ├── src/main/java/com/devsuperior/dsvendas/
│   │   ├── config/          # Configuração de segurança e CORS
│   │   ├── controllers/     # Endpoints REST (Sale, Seller)
│   │   ├── dto/              # DTOs expostos pela API
│   │   ├── entities/         # Entidades JPA (Sale, Seller)
│   │   ├── repositories/     # Spring Data JPA repositories
│   │   └── service/          # Regras de negócio
│   ├── src/main/resources/   # application*.properties, data.sql
│   ├── src/test/             # Testes automatizados
│   └── pom.xml
├── frontend/
│   ├── src/
│   │   ├── components/       # BarChart, DonoutChart, DataTable, Pagination, NavBar, Footer
│   │   ├── pages/             # Home, Dashboard
│   │   ├── types/              # Tipos TypeScript (Sale, Seller)
│   │   └── utils/               # Formatação e helpers de requisição
│   └── package.json
├── docs/images/               # Screenshots da aplicação (ver README da pasta)
└── LICENSE
```

## Segurança

Durante a preparação deste README, os seguintes pontos de segurança foram identificados e tratados:

- **Credencial de banco hardcoded** — `backend/src/main/resources/application-dev.properties` continha usuário e senha de PostgreSQL fixos no código-fonte. Isso foi corrigido: as credenciais agora vêm de variáveis de ambiente (`DB_URL`, `DB_USERNAME`, `DB_PASSWORD`), com um `backend/.env.example` documentando os placeholders, e `.env` foi adicionado ao `.gitignore`.
  - **Risco residual:** a senha antiga (`123456`, válida apenas para uma instância local de desenvolvimento) permanece no histórico de commits do Git. Reescrever o histórico está fora do escopo desta correção; se essa senha tiver sido reutilizada em algum ambiente real, ela deve ser rotacionada manualmente.
- **API sem autenticação** — todos os endpoints (`/sales`, `/sales/amount-by-seller`, `/sales/success-by-seller`, `/sellers`) são públicos (`permitAll()` em `SecurityConfig`). **Isso não foi alterado nesta correção** pelos seguintes motivos, e continua como item prioritário do roadmap abaixo:
  1. Não existe, em nenhum lugar do código, qualquer mecanismo de autenticação já montado (não há entidade de usuário, login, JWT ou sessão) — apenas a dependência do Spring Security, usada hoje só para CORS/CSRF/stateless session.
  2. O front end não possui nenhuma tela de login ou tratamento de token; adicionar exigência de autenticação na API sem uma contraparte no front end quebraria a aplicação que está no ar hoje.
  3. Construir um sistema de autenticação do zero (entidade de usuário, hashing de senha, emissão/validação de JWT, tela de login) é uma mudança arquitetural grande, fora do escopo de uma correção pontual — e o risco de introduzir regressões sem testes de regressão robustos é alto.

## Roadmap / melhorias futuras

- [ ] **[Segurança — prioridade alta]** Implementar autenticação (por exemplo, Spring Security + JWT) protegendo os endpoints da API, junto com uma tela de login no front end. Hoje toda a API é pública.
- [ ] **[Segurança]** Rotacionar a credencial de banco que ficou exposta no histórico do Git (`123456` em `application-dev.properties`), caso tenha sido reutilizada fora do ambiente local.
- [ ] Adicionar testes unitários e de integração para os `Controllers`, `Services` e `Repositories` do back end (hoje há apenas o teste de contexto).
- [ ] Adicionar testes de front end (React Testing Library já está instalado como dependência, mas nenhum teste foi escrito).
- [ ] Avaliar a migração para Spring Boot 3.x (requer migrar de `javax.*` para `jakarta.*` e substituir `WebSecurityConfigurerAdapter`, hoje depreciado, pela configuração baseada em `SecurityFilterChain`). Não foi feita nesta correção por ser uma mudança arquitetural maior, sem testes automatizados suficientes para validar a migração com segurança; o bump feito aqui (2.4.5 → 2.7.18) já remove o CVE mais crítico de versões antigas do Spring Boot sem exigir essa migração.
- [ ] Adicionar paginação/filtro por data na listagem de vendas.
- [ ] Adicionar screenshots reais em `docs/images/` (ver `docs/images/README.md`).

## Contribuição

Contribuições são bem-vindas.

1. Faça um fork do repositório.
2. Crie uma branch a partir de `master`: `git checkout -b minha-feature`.
3. Faça commits pequenos e descritivos.
4. Rode os testes (`./mvnw test` no back end, `npm test` no front end) antes de abrir o Pull Request.
5. Abra um Pull Request descrevendo a mudança.

## Licença

Este projeto está licenciado sob a licença MIT — veja o arquivo [LICENSE](./LICENSE) para mais detalhes.
