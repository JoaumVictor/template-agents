# Padrão de Projeto Plathanus

> Documento de referência baseado na arquitetura e boas práticas do projeto CAARS.
> Use este guia como ponto de partida para novos projetos.

---

## Índice

1. [Visão Geral da Arquitetura](#1-visão-geral-da-arquitetura)
2. [Mobile — React Native + Expo](#2-mobile--react-native--expo)
3. [Frontend Web — Admin/Dashboard](#3-frontend-web--admindashboard)
4. [Frontend Web — Público/Marketing](#4-frontend-web--públicomarketing)
5. [Backend — Microserviços NestJS](#5-backend--microserviços-nestjs)
6. [Banco de Dados — PostgreSQL + TypeORM](#6-banco-de-dados--postgresql--typeorm)
7. [Cache — Redis](#7-cache--redis)
8. [Mensageria — RabbitMQ](#8-mensageria--rabbitmq)
9. [Infraestrutura — Docker + Docker Swarm](#9-infraestrutura--docker--docker-swarm)
10. [CI/CD — GitHub Actions](#10-cicd--github-actions)
11. [Observabilidade — Prometheus, Grafana, Loki](#11-observabilidade--prometheus-grafana-loki)
12. [Autenticação e Segurança](#12-autenticação-e-segurança)
13. [Estrutura de Monorepo](#13-estrutura-de-monorepo)
14. [Variáveis de Ambiente](#14-variáveis-de-ambiente)
15. [Boas Práticas Gerais](#15-boas-práticas-gerais)

---

## 1. Visão Geral da Arquitetura

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENTES                             │
│   Mobile (RN+Expo)  |  Admin (React)  |  Web (Next.js)     │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTPS
                    ┌──────▼──────┐
                    │   HAProxy   │  ← Edge / Load Balancer
                    └──────┬──────┘
                           │
                    ┌──────▼──────┐
                    │ API Gateway │  ← Ponto de entrada único (porta 3000)
                    └──────┬──────┘
                           │ HTTP interno
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐
   │  Identity   │  │  Benefits   │  │ Engagement  │  ← Microserviços NestJS
   │  Service    │  │  Catalog    │  │  Comms      │
   └──────┬──────┘  └──────┬──────┘  └──────┬──────┘
          │                │                │
   ┌──────▼──────┐  ┌──────▼──────┐  ┌──────▼──────┐
   │ PostgreSQL  │  │ PostgreSQL  │  │ PostgreSQL  │  ← DB por serviço
   └─────────────┘  └─────────────┘  └─────────────┘
          │
   ┌──────▼──────┐
   │   Redis     │  ← Cache distribuído
   └─────────────┘
          │
   ┌──────▼──────┐
   │  RabbitMQ   │  ← Mensageria assíncrona (eventos entre serviços)
   └─────────────┘
```

**Princípios fundamentais:**

- Um API Gateway como porta de entrada única
- Cada serviço tem seu próprio banco de dados (database-per-service)
- Comunicação assíncrona via eventos no RabbitMQ
- Comunicação síncrona via HTTP interno somente pelo gateway
- Nenhum serviço chama outro serviço diretamente via REST

---

## 2. Mobile — React Native + Expo

### Stack

| Camada               | Tecnologia                   | Versão           |
| -------------------- | ---------------------------- | ---------------- |
| Framework            | React Native + Expo          | 0.83.x + Expo 55 |
| Linguagem            | TypeScript                   | 5.9+             |
| Roteamento           | Expo Router (file-based)     | 55.0             |
| Estado global        | Zustand                      | 5.0              |
| Estado servidor      | React Query (TanStack)       | 5.x              |
| HTTP                 | Axios                        | 1.7+             |
| Estilização          | NativeWind (Tailwind)        | 4.0              |
| Armazenamento seguro | Expo SecureStore             | 55.0             |
| Animações            | React Native Reanimated      | 4.x              |
| Gestos               | react-native-gesture-handler | 2.x              |

### Estrutura de pastas

```
client/app/
├── app/                        # Expo Router — cada arquivo = uma rota
│   ├── _layout.tsx             # Layout raiz (providers globais aqui)
│   ├── (auth)/                 # Grupo de rotas sem autenticação
│   │   ├── _layout.tsx
│   │   ├── login.tsx
│   │   └── forgot-password.tsx
│   └── (app)/                  # Grupo de rotas autenticadas
│       ├── _layout.tsx         # Tab navigator ou drawer aqui
│       ├── home/
│       │   └── index.tsx
│       ├── profile/
│       │   ├── index.tsx
│       │   └── [id].tsx        # Rota dinâmica
│       └── notifications/
│           └── index.tsx
├── components/                 # Componentes reutilizáveis (PascalCase)
│   ├── ui/                     # Componentes base (Button, Input, Card)
│   └── [feature]/              # Componentes de domínio
├── hooks/                      # Custom hooks (useXxx)
├── store/                      # Zustand stores
├── api/                        # Axios client + chamadas por domínio
├── types/                      # Interfaces TypeScript
├── lib/                        # Funções utilitárias
├── assets/                     # Imagens, ícones, fontes
├── app.config.js               # Configuração do Expo (nome, ícone, permissões)
└── [.env.ios / .env.android]   # Envs por plataforma
```

### Padrões de código

**Store Zustand:**

```typescript
// store/auth.store.ts
interface AuthState {
  token: string | null;
  user: User | null;
  setToken: (token: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  token: null,
  user: null,
  setToken: (token) => set({ token }),
  logout: () => set({ token: null, user: null }),
}));
```

**React Query + Axios:**

```typescript
// api/campaigns.api.ts
export const getCampaigns = () =>
  api.get<Campaign[]>("/campaigns").then((r) => r.data);

// hooks/useCampaigns.ts
export const useCampaigns = () =>
  useQuery({ queryKey: ["campaigns"], queryFn: getCampaigns });
```

**Axios com JWT automático:**

```typescript
// api/client.ts
api.interceptors.request.use(async (config) => {
  const token = await SecureStore.getItemAsync("token");
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});
```

### Boas práticas mobile

- Nunca armazenar JWT no AsyncStorage — sempre usar **SecureStore**
- Separar rotas `(auth)` e `(app)` em grupos para controle de acesso
- Usar `_layout.tsx` para injetar providers (QueryClient, Zustand, ThemeProvider)
- Variáveis de ambiente por plataforma: `.env.ios`, `.env.android`
- Usar `React.memo` e `useCallback` em listas longas (FlatList)
- Preferir `NativeWind` ao StyleSheet para consistência com o admin web

---

## 3. Frontend Web — Admin/Dashboard

### Stack

| Camada          | Tecnologia             | Versão     |
| --------------- | ---------------------- | ---------- |
| Framework       | React + Vite           | 18.3 + 5.4 |
| Linguagem       | TypeScript             | 5.6+       |
| Roteamento      | React Router DOM       | 6.x        |
| Estado global   | Zustand                | 5.0        |
| Estado servidor | React Query (TanStack) | 5.x        |
| HTTP            | Axios                  | 1.7+       |
| UI              | Radix UI + shadcn/ui   | latest     |
| Estilização     | Tailwind CSS           | 3.4+       |
| Formulários     | React Hook Form + Zod  | 7.x + 3.x  |
| Gráficos        | Recharts               | 2.x        |
| Ícones          | Lucide React           | latest     |

### Estrutura de pastas

```
client/admin/src/
├── pages/                      # Uma pasta por rota principal
│   ├── home/
│   │   └── Home.tsx
│   └── campaigns/
│       ├── CampaignList.tsx
│       └── CampaignDetail.tsx
├── components/                 # Componentes reutilizáveis
│   ├── ui/                     # Primitivos (Button, Input, Modal, Table)
│   └── layout/                 # Header, Sidebar, PageWrapper
├── hooks/                      # Custom hooks
├── api/                        # Axios client + funções por domínio
├── store/                      # Zustand stores
├── types/                      # Interfaces TypeScript
├── lib/                        # Helpers, formatadores
├── config/                     # Constantes, configurações de app
├── assets/                     # Imagens, SVGs
└── router.tsx                  # Definição de rotas (React Router)
```

### Padrões de código

**Roteamento:**

```typescript
// router.tsx
export const router = createBrowserRouter([
  {
    path: '/',
    element: <PrivateRoute />,
    children: [
      { index: true, element: <Home /> },
      { path: 'campaigns', element: <CampaignList /> },
    ],
  },
  { path: '/login', element: <Login /> },
])
```

**Formulário com Zod:**

```typescript
const schema = z.object({
  name: z.string().min(3),
  email: z.string().email(),
});

const { register, handleSubmit } = useForm<z.infer<typeof schema>>({
  resolver: zodResolver(schema),
});
```

### Build e Deploy

- Build: `vite build` → pasta `dist/` (SPA)
- Servido por **Nginx Alpine** via Docker
- Path aliases: `@/*` mapeia para `src/*`

---

## 4. Frontend Web — Público/Marketing

### Stack

| Camada      | Tecnologia                 |
| ----------- | -------------------------- |
| Framework   | Next.js 14 (App Router)    |
| Linguagem   | TypeScript                 |
| Estilização | Tailwind CSS               |
| Build       | `next build` → Node server |

### Quando usar Next.js vs Vite+React

| Critério               | Next.js | Vite + React |
| ---------------------- | ------- | ------------ |
| SEO necessário         | ✅      | ❌           |
| Página pública/landing | ✅      | ❌           |
| SSR/SSG necessário     | ✅      | ❌           |
| App interno/admin      | ❌      | ✅           |
| Menor complexidade     | ❌      | ✅           |

### Estrutura de pastas

```
client/public-web/
├── app/                        # App Router
│   ├── layout.tsx
│   ├── page.tsx                # Home
│   └── [slug]/
│       └── page.tsx
├── components/
├── public/                     # Assets estáticos
└── next.config.js
```

---

## 5. Backend — Microserviços NestJS

### Stack

| Camada      | Tecnologia                          | Versão             |
| ----------- | ----------------------------------- | ------------------ |
| Framework   | NestJS                              | 10.4+              |
| Linguagem   | TypeScript                          | 5.6+ (strict mode) |
| ORM         | TypeORM                             | 0.3.x              |
| Auth        | Passport + JWT + bcrypt             | latest             |
| Validação   | class-validator + class-transformer | 0.14+              |
| Config      | Joi (validação de env)              | 17.x               |
| HTTP Client | Axios                               | 1.15+              |
| RabbitMQ    | amqplib + amqp-connection-manager   | latest             |
| Email       | Nodemailer                          | 8.x                |

### Estrutura de um serviço

```
backend/[nome]-service/
├── src/
│   ├── main.ts                 # Bootstrap (porta, CORS, pipes globais)
│   ├── app.module.ts           # Módulo raiz (imports, ConfigModule)
│   ├── data-source.ts          # DataSource do TypeORM (usado nas migrations)
│   │
│   ├── [domínio]/              # Módulo de feature (ex: auth, campaigns)
│   │   ├── [domínio].module.ts
│   │   ├── [domínio].controller.ts
│   │   ├── [domínio].service.ts
│   │   ├── dto/
│   │   │   ├── create-[domínio].dto.ts
│   │   │   └── update-[domínio].dto.ts
│   │   └── entities/
│   │       └── [domínio].entity.ts
│   │
│   ├── common/                 # Compartilhado por todos os módulos
│   │   ├── guards/             # AuthGuard, RolesGuard
│   │   ├── decorators/         # @CurrentUser(), @Roles()
│   │   ├── interceptors/       # TransformInterceptor, LoggingInterceptor
│   │   ├── filters/            # HttpExceptionFilter
│   │   └── pipes/              # ValidationPipe
│   │
│   ├── messaging/              # Handlers de eventos RabbitMQ
│   │   └── [evento].handler.ts
│   │
│   └── migrations/             # Migrations TypeORM
│       └── [timestamp]-init.ts
│
├── Dockerfile
├── package.json
└── tsconfig.json
```

### Padrões de código

**Controller:**

```typescript
@Controller("campaigns")
@UseGuards(JwtAuthGuard)
export class CampaignsController {
  constructor(private readonly campaignsService: CampaignsService) {}

  @Get()
  findAll(@CurrentUser() user: AuthUser) {
    return this.campaignsService.findAll(user.id);
  }

  @Post()
  create(@Body() dto: CreateCampaignDto, @CurrentUser() user: AuthUser) {
    return this.campaignsService.create(dto, user.id);
  }
}
```

**DTO com validação:**

```typescript
export class CreateCampaignDto {
  @IsString()
  @MinLength(3)
  name: string;

  @IsDateString()
  startsAt: string;

  @IsOptional()
  @IsNumber()
  discount?: number;
}
```

**Entity TypeORM:**

```typescript
@Entity("campaigns")
export class Campaign {
  @PrimaryGeneratedColumn("uuid")
  id: string;

  @Column()
  name: string;

  @Column({ type: "timestamp" })
  startsAt: Date;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

**Validação de ENV com Joi no AppModule:**

```typescript
@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: Joi.object({
        DATABASE_URL: Joi.string().required(),
        JWT_SECRET: Joi.string().required(),
        PORT: Joi.number().default(3001),
      }),
    }),
  ],
})
export class AppModule {}
```

### API Gateway

- Porta única exposta externamente: **3000**
- Faz proxy para os serviços internos via nome de serviço Docker
- Responsabilidades: rate limiting, CORS, logging de entrada
- **Não** contém lógica de negócio

### Boas práticas backend

- Sempre validar o ambiente com Joi no startup — falhe cedo se faltar ENV
- DTOs obrigatórios em todos os endpoints que recebem body
- Guards globais no `main.ts`, não em cada controller
- Usar `ConfigService` (NestJS) em vez de acessar `process.env` diretamente
- Separar lógica de negócio em `Service`, nunca no `Controller`
- Usar `@CurrentUser()` decorator para acessar o usuário autenticado

---

## 6. Banco de Dados — PostgreSQL + TypeORM

### Estratégia: Database-per-Service

Cada microserviço tem seu **próprio banco de dados**. Serviços nunca compartilham tabelas.

| Serviço            | Banco           | Porta dev |
| ------------------ | --------------- | --------- |
| identity-service   | `db_identity`   | 5432      |
| benefits-service   | `db_benefits`   | 5433      |
| experience-service | `db_experience` | 5434      |
| engagement-service | `db_engagement` | 5435      |

### Configuração TypeORM

```typescript
// src/data-source.ts
export const AppDataSource = new DataSource({
  type: "postgres",
  url: process.env.DATABASE_URL,
  entities: [__dirname + "/**/*.entity{.ts,.js}"],
  migrations: [__dirname + "/migrations/*{.ts,.js}"],
  synchronize: false, // NUNCA true em produção
});
```

### Migrations

```bash
# Gerar migration com base nas entidades
npm run typeorm -- migration:generate src/migrations/NomeDaAlteracao

# Rodar migrations pendentes
npm run typeorm -- migration:run

# Reverter última migration
npm run typeorm -- migration:revert
```

**Script no package.json:**

```json
{
  "scripts": {
    "typeorm": "typeorm-ts-node-commonjs -d src/data-source.ts"
  }
}
```

### HA em Produção

```
PostgreSQL Primary (leitura + escrita)
         │
         └── Hot Standby (replicação contínua via WAL)
                │
                └── Failover automático (Patroni) ou manual (runbook)
```

- RPO (perda máxima de dados): ≤ 5 minutos
- RTO (tempo de recuperação): ≤ 10 min (automático) / ≤ 30 min (manual)

---

## 7. Cache — Redis

### Quando usar

- Sessões de usuário
- Rate limiting
- Cache de queries caras (campanhas ativas, catálogos)
- Dados transitórios (OTP, tokens de recuperação)

### Setup no NestJS

```typescript
// app.module.ts
CacheModule.register({
  isGlobal: true,
  store: redisStore,
  host: process.env.REDIS_HOST,
  port: Number(process.env.REDIS_PORT),
  ttl: 60, // segundos
});
```

**Acesso direto com ioredis:**

```typescript
const redis = new Redis(process.env.REDIS_URL);
await redis.set("key", value, "EX", 300); // TTL 5 min
const cached = await redis.get("key");
```

---

## 8. Mensageria — RabbitMQ

### Arquitetura de eventos

```
Serviço A publica → Exchange "projeto.events" (topic) → Queue → Serviço B consome
```

### Quando usar RabbitMQ vs chamada HTTP direta

| Cenário                                | RabbitMQ | HTTP direto      |
| -------------------------------------- | -------- | ---------------- |
| Notificar outros serviços de um evento | ✅       | ❌               |
| Operação assíncrona (email, push)      | ✅       | ❌               |
| Resposta imediata necessária           | ❌       | ✅               |
| Busca de dado de outro serviço         | ❌       | ✅ (via gateway) |

### Padrão de Publisher

```typescript
// messaging/events.publisher.ts
@Injectable()
export class EventsPublisher {
  constructor(private readonly amqp: AmqpConnection) {}

  async publish(routingKey: string, payload: object) {
    await this.amqp.publish("projeto.events", routingKey, payload);
  }
}
```

### Padrão de Consumer

```typescript
// messaging/user-created.handler.ts
@Injectable()
export class UserCreatedHandler {
  @RabbitSubscribe({
    exchange: "projeto.events",
    routingKey: "user.created",
    queue: "benefits.user-created",
  })
  async handle(payload: { userId: string; email: string }) {
    // inicializar conta do usuário no serviço de benefícios
  }
}
```

### Convenções de routing keys

```
[serviço-origem].[entidade].[evento]

Exemplos:
  identity.user.created
  identity.user.deleted
  benefits.payment.completed
  benefits.campaign.activated
```

---

## 9. Infraestrutura — Docker + Docker Swarm

### Desenvolvimento local

Um único `docker-compose.yml` na raiz sobe tudo:

```yaml
services:
  api-gateway:
    build: ./backend/api-gateway
    ports: ["3000:3000"]
    depends_on: [db_identity, rabbitmq, redis]

  identity-service:
    build: ./backend/identity-service
    depends_on: [db_identity]

  db_identity:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: identity_db
      POSTGRES_PASSWORD: senha_local
    volumes: [identity_data:/var/lib/postgresql/data]

  redis:
    image: redis:7-alpine

  rabbitmq:
    image: rabbitmq:3.13-management-alpine
    ports: ["5672:5672", "15672:15672"]
```

```bash
docker-compose up -d       # Sobe tudo
docker-compose logs -f     # Acompanha logs
docker-compose down -v     # Para e limpa volumes
```

### Dockerfile padrão (backend NestJS)

```dockerfile
# Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM node:20-alpine AS runner
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY --from=builder /app/dist ./dist
EXPOSE 3001
CMD ["node", "dist/main.js"]
```

### Dockerfile padrão (frontend React/Vite → Nginx)

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM nginx:alpine AS runner
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### Produção — Docker Swarm

**Topologia de stacks:**

```
┌─────────────────────────────────────────────────┐
│ Stack: edge                                      │
│   HAProxy (global, portas 80/443)                │
│   Keepalived VRRP (Virtual IP — systemd)         │
└─────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────┐
│ Stack: app                                       │
│   api-gateway (2 réplicas)                       │
│   identity-service (2 réplicas)                  │
│   benefits-service (2 réplicas)                  │
│   ... (rolling update, 1 por vez, 10s delay)     │
└─────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────┐
│ Stack: data                                      │
│   PostgreSQL x4 (primary por DB)                 │
│   Redis                                          │
│   RabbitMQ                                       │
└─────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────┐
│ Stack: ops                                       │
│   Prometheus + Grafana + Loki + Promtail         │
└─────────────────────────────────────────────────┘
```

**Estratégia de deploy (rolling update):**

```yaml
deploy:
  replicas: 2
  update_config:
    parallelism: 1
    delay: 10s
    order: start-first
    failure_action: rollback
  rollback_config:
    parallelism: 1
    delay: 5s
```

**Deploy de uma stack:**

```bash
docker stack deploy -c infra/app/docker-compose.app.yml app --with-registry-auth
docker stack ls
docker service ls
```

### Estrutura de infra

```
infra/
├── app/
│   └── docker-compose.app.yml      # Serviços da aplicação
├── data/
│   └── docker-compose.data.yml     # Bancos, Redis, RabbitMQ
├── edge-ha/
│   ├── docker-compose.edge.yml     # HAProxy
│   ├── haproxy/                    # haproxy.cfg
│   └── keepalived/                 # VRRP config
├── ops/
│   ├── docker-compose.observability.yml
│   ├── monitoring/prometheus/
│   └── logging/loki/ + promtail/
└── runbooks/                       # Procedimentos operacionais
    ├── swarm-deploy.md
    ├── disaster-recovery-data.md
    └── failover-edge.md
```

---

## 10. CI/CD — GitHub Actions

### Fluxo

```
Push para main
      │
      ▼
validate-infra          ← Valida sintaxe dos docker-compose.yml
      │
      ▼
deploy-production       ← Self-hosted runner (Linux, no Swarm manager)
  ├── Carrega .env.production (secret base64)
  ├── docker stack deploy data
  ├── docker stack deploy app
  └── docker stack deploy ops
```

### Template de workflow

```yaml
# .github/workflows/deploy.yml
name: Deploy Production

on:
  push:
    branches: [main]

jobs:
  validate-infra:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Validate compose files
        run: |
          docker compose -f infra/data/docker-compose.data.yml config -q
          docker compose -f infra/app/docker-compose.app.yml config -q

  deploy-production:
    needs: validate-infra
    runs-on: [self-hosted, linux, swarm, prod]
    steps:
      - uses: actions/checkout@v4
      - name: Prepare env
        run: echo "${{ secrets.PROD_ENV_FILE_B64 }}" | base64 -d > .env.production
      - name: Deploy stacks
        run: |
          docker stack deploy -c infra/data/docker-compose.data.yml data
          docker stack deploy -c infra/app/docker-compose.app.yml app
          docker stack deploy -c infra/ops/docker-compose.observability.yml ops
```

### Secrets necessários

| Secret                   | Descrição                         |
| ------------------------ | --------------------------------- |
| `PROD_ENV_FILE_B64`      | `.env.production` em base64       |
| `PROD_REGISTRY_URL`      | URL do registry Docker (opcional) |
| `PROD_REGISTRY_USERNAME` | Usuário do registry (opcional)    |
| `PROD_REGISTRY_PASSWORD` | Senha do registry (opcional)      |

**Gerar o secret de env:**

```bash
base64 -w 0 .env.production | pbcopy  # macOS
base64 -w 0 .env.production | clip    # Windows
```

---

## 11. Observabilidade — Prometheus, Grafana, Loki

### Stack de observabilidade

```
Serviços NestJS → Prometheus (métricas)
                       │
                       └── Grafana (dashboards)

Containers Docker  → Promtail (coleta logs)
                          │
                          └── Loki (armazenamento)
                                │
                                └── Grafana (consulta logs)
```

### Expor métricas no NestJS

```bash
npm install @willsoto/nestjs-prometheus prom-client
```

```typescript
// app.module.ts
PrometheusModule.register({
  defaultMetrics: { enabled: true },
  path: "/metrics",
});
```

### Configuração Prometheus

```yaml
# infra/ops/monitoring/prometheus/prometheus.yml
scrape_configs:
  - job_name: api-gateway
    static_configs:
      - targets: ["api-gateway:3000"]
    metrics_path: /metrics

  - job_name: identity-service
    static_configs:
      - targets: ["identity-service:3001"]
```

### Configuração Promtail (coleta de logs Docker)

```yaml
# infra/ops/logging/promtail/config.yml
scrape_configs:
  - job_name: docker
    docker_sd_configs:
      - host: unix:///var/run/docker.sock
    relabel_configs:
      - source_labels: [__meta_docker_container_name]
        target_label: container
```

### O que monitorar

| Métrica                  | Ferramenta          | Alerta sugerido              |
| ------------------------ | ------------------- | ---------------------------- |
| Latência HTTP (p95, p99) | Prometheus          | > 500ms                      |
| Taxa de erros (5xx)      | Prometheus          | > 1%                         |
| Uso de CPU/memória       | Prometheus          | > 80%                        |
| Conexões de banco        | Prometheus          | > 80% do pool                |
| Filas RabbitMQ           | RabbitMQ Management | > 1000 msgs                  |
| Logs de erro             | Loki                | qualquer `ERROR` em produção |

---

## 12. Autenticação e Segurança

### Fluxo JWT

```
Login → bcrypt.compare(senha) → jwt.sign(payload) → Bearer token
                                                           │
GET /rota → JwtAuthGuard → jwt.verify() → @CurrentUser() ─┘
```

### Setup no NestJS

```typescript
// auth.module.ts
JwtModule.registerAsync({
  useFactory: (config: ConfigService) => ({
    secret: config.get("JWT_SECRET"),
    signOptions: { expiresIn: "7d" },
  }),
  inject: [ConfigService],
});
```

**Guard:**

```typescript
@Injectable()
export class JwtAuthGuard extends AuthGuard("jwt") {}
```

**Decorator @CurrentUser:**

```typescript
export const CurrentUser = createParamDecorator(
  (_, ctx: ExecutionContext) => ctx.switchToHttp().getRequest().user,
);
```

### Boas práticas de segurança

- Sempre usar `bcrypt` (rounds ≥ 12) para senhas
- JWT com expiração curta para access token (15min–1h)
- Refresh token em cookie httpOnly se necessário
- Nunca logar senhas, tokens ou dados sensíveis
- Validar e sanitizar TODA entrada de usuário com class-validator
- CORS configurado explicitamente (não `*` em produção)
- Rate limiting no API Gateway (ex: 100 req/min por IP)
- Variáveis de ambiente para segredos — nunca hardcoded

---

## 13. Estrutura de Monorepo

### Organização

```
projeto/
├── backend/                    # Microserviços (cada um independente)
│   ├── api-gateway/
│   ├── [nome]-service/
│   └── [nome]-service/
├── client/                     # Frontends
│   ├── app/                    # Mobile
│   ├── admin/                  # Dashboard web
│   └── [nome]-web/             # Site público (Next.js)
├── infra/                      # Infraestrutura e DevOps
│   ├── app/
│   ├── data/
│   ├── edge-ha/
│   ├── ops/
│   └── runbooks/
├── docs/                       # Documentação técnica viva
├── .github/workflows/          # CI/CD
├── docker-compose.yml          # Dev local (todos os serviços)
├── .env.example                # Template de env
└── CLAUDE.md                   # Instruções para agentes de IA
```

**Princípio:** Cada serviço é autossuficiente — tem seu `package.json`, `tsconfig.json` e `Dockerfile` próprios. Não há workspaces compartilhados.

### Quando adicionar um novo serviço

1. Criar pasta em `backend/[nome]-service/`
2. Usar estrutura padrão NestJS descrita na seção 5
3. Adicionar ao `docker-compose.yml` (dev) com porta nova
4. Adicionar ao `infra/app/docker-compose.app.yml` (produção)
5. Adicionar banco em `infra/data/docker-compose.data.yml`
6. Adicionar proxy no API Gateway
7. Documentar em `docs/`

---

## 14. Variáveis de Ambiente

### Organização

| Arquivo                   | Uso                                              |
| ------------------------- | ------------------------------------------------ |
| `.env.example`            | Template completo para dev — commitar este       |
| `.env`                    | Valores locais — NÃO commitar                    |
| `.env.production.example` | Template para produção — commitar                |
| `.env.production`         | Valores reais — NÃO commitar (usar secret no CI) |
| `client/app/.env.ios`     | ENV específico para iOS build                    |
| `client/app/.env.android` | ENV específico para Android build                |

### Variáveis padrão por serviço

```bash
# Comum a todos os serviços backend
NODE_ENV=development
PORT=3001

# Banco de dados
DATABASE_URL=postgres://user:pass@localhost:5432/db_name

# JWT (somente no serviço de identidade)
JWT_SECRET=segredo-super-secreto
JWT_EXPIRES_IN=7d

# Redis (serviços que usam cache)
REDIS_URL=redis://localhost:6379

# RabbitMQ (serviços que publicam/consomem eventos)
RABBITMQ_URL=amqp://guest:guest@localhost:5672

# API Gateway
IDENTITY_SERVICE_URL=http://identity-service:3001
BENEFITS_SERVICE_URL=http://benefits-service:3002
```

### Validação obrigatória de ENV

Todo serviço deve validar as ENVs no startup com Joi. Se uma ENV obrigatória estiver faltando, a aplicação deve **falhar ao iniciar** com mensagem clara.

```typescript
validationSchema: Joi.object({
  DATABASE_URL: Joi.string().required(),
  JWT_SECRET: Joi.string().min(32).required(),
  PORT: Joi.number().default(3001),
});
```

---

## 15. Boas Práticas Gerais

### TypeScript

- Sempre `strict: true` no tsconfig
- Nunca usar `any` — usar `unknown` quando necessário e fazer type guard
- Tipos explícitos em retornos de funções públicas
- Preferir `interface` para contratos, `type` para uniões/utilitários

### Git e Versionamento

```
feat: adiciona campanha de cashback
fix: corrige cálculo de desconto
refactor: extrai validação de CPF para helper
chore: atualiza dependências
docs: adiciona runbook de failover
```

- Branch por feature: `feature/nome-da-feature`
- PRs pequenos e focados — um assunto por PR
- Nunca commitar `.env`, segredos ou binários grandes

### Nomenclatura

| Item                  | Convenção                       | Exemplo                   |
| --------------------- | ------------------------------- | ------------------------- |
| Arquivos TypeScript   | kebab-case                      | `user-profile.service.ts` |
| Classes               | PascalCase                      | `UserProfileService`      |
| Variáveis/funções     | camelCase                       | `getUserById`             |
| Constantes globais    | UPPER_SNAKE_CASE                | `MAX_RETRY_COUNT`         |
| Tabelas no banco      | snake_case                      | `user_profiles`           |
| Colunas no banco      | snake_case                      | `created_at`              |
| Routing keys RabbitMQ | `[serviço].[entidade].[evento]` | `identity.user.created`   |
| Stacks Docker         | kebab-case                      | `app`, `data`, `ops`      |

### Documentação mínima

Todo projeto deve ter:

- `README.md` na raiz com: o que é, como rodar localmente, como fazer deploy
- `.env.example` com todas as variáveis necessárias comentadas
- `infra/runbooks/` com procedimentos de deploy, failover e recovery
- `CLAUDE.md` com instruções para agentes de IA (se aplicável)

### O que NÃO fazer

- Não compartilhar banco de dados entre serviços
- Não chamar serviços internos diretamente (sempre via gateway em prod)
- Não usar `synchronize: true` no TypeORM em produção
- Não commitar segredos ou `.env` com valores reais
- Não usar `docker-compose down -v` em produção (apaga volumes)
- Não fazer deploy direto em main sem CI validando
- Não logar informações sensíveis (senhas, tokens, CPF, cartão)

---

## Referência rápida — Tecnologias por camada

| Camada            | Tecnologia principal                          | Auxiliares                                       |
| ----------------- | --------------------------------------------- | ------------------------------------------------ |
| Mobile            | React Native 0.83 + Expo 55                   | Expo Router, NativeWind, SecureStore, Reanimated |
| Web Admin         | React 18 + Vite 5                             | React Router 6, shadcn/ui, Tailwind              |
| Web Público       | Next.js 14                                    | Tailwind                                         |
| Backend           | NestJS 10 + TypeScript                        | TypeORM, Passport, class-validator               |
| Banco de Dados    | PostgreSQL 16                                 | TypeORM, migrations                              |
| Cache             | Redis 7                                       | ioredis                                          |
| Filas             | RabbitMQ 3.13                                 | amqplib, amqp-connection-manager                 |
| Estado (frontend) | Zustand 5 (global) + React Query 5 (servidor) | —                                                |
| Formulários       | React Hook Form 7 + Zod 3                     | —                                                |
| Containers (dev)  | Docker Compose                                | —                                                |
| Containers (prod) | Docker Swarm                                  | HAProxy, Keepalived                              |
| CI/CD             | GitHub Actions                                | Self-hosted runner                               |
| Métricas          | Prometheus + Grafana                          | prom-client                                      |
| Logs              | Loki + Promtail + Grafana                     | —                                                |
| Auth              | JWT + bcrypt                                  | Passport, NestJS JWT                             |
| Testes de carga   | k6                                            | —                                                |
