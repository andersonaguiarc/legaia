# ADR-003: Estrategia de Migración a Microservicios

**Fecha:** 27 de Enero, 2026  
**Status:** ⏸️ **Deferred** (Para futuro, cuando se cumplan triggers)  
**Decisor:** Anderson Aguiar (CTO)  
**Stakeholders:** Katherine Aguiar (CLO), María Juliana Grajales (CLO)

---

## Context

LegaIA inicia como **Monolito Modular** (ver ADR-001), pero eventualmente puede necesitar migrar a microservicios. Este ADR define:
1. **CUÁNDO** migrar (triggers específicos)
2. **CÓMO** migrar (estrategia Strangler Pattern)
3. **QUÉ** migrar primero (priorización de servicios)
4. **POR QUÉ** algunos módulos se quedan en monolito

### Contexto Actual (2026)

```
LegaIA Estado Actual:
├─ Arquitectura: Monolito modular
├─ Usuarios: MVP (target 280 Year 1)
├─ Team: 1 dev (Anderson)
├─ Revenue: $0 (pre-launch)
├─ Tráfico: 0 requests/día
└─ Costo infra: $21-51/mes
```

### Visión Futura (Hipotética)

```
LegaIA Año 3-5:
├─ Arquitectura: Híbrido (monolito + microservicios selectivos)
├─ Usuarios: 5K-50K
├─ Team: 5-10 devs
├─ Revenue: $500K-2M/año
├─ Tráfico: 100K-500K requests/día
└─ Costo infra: $5K-20K/mes
```

---

## Decision

**NO migrar a microservicios inmediatamente.** En su lugar:

1. **Mantener monolito modular** hasta que se cumplan triggers específicos
2. **Usar Strangler Pattern** para migración gradual (no big bang)
3. **Extraer servicios selectivamente** (no todo)
4. **Mantener core en monolito** indefinidamente

---

## Triggers para Migración

### ❌ **DO NOT MIGRATE** si:

```
Usuarios < 1000 activos/día
AND
Revenue < $500K/año
AND
Team < 3 devs
AND
Un módulo NO necesita escalar 10x independiente
```

### ✅ **CONSIDER MIGRATION** si se cumplen 2+ de:

#### Trigger 1: Escalabilidad Diferencial Crítica

```
SÍNTOMA:
Un módulo específico necesita 10x más recursos que el resto.

EJEMPLO:
├─ AI Generation: 1000 requests/min, 4GB RAM, 2 CPU
├─ Auth: 50 requests/min, 256MB RAM, 0.2 CPU
├─ Templates: 20 requests/min, 128MB RAM, 0.1 CPU
└─ Payments: 100 requests/min, 512MB RAM, 0.3 CPU

PROBLEMA:
Escalar todo el monolito para AI = desperdiciar recursos
en auth, templates, payments.

COSTO:
Monolito escalado: $800/mes
Microservicios: $400/mes (solo AI escala)

TRIGGER: ✅ Migrar AI Generation Service
```

#### Trigger 2: Equipos Independientes Bloqueados

```
SÍNTOMA:
3+ devs haciendo merge conflicts frecuentes en mismo repo.

EJEMPLO:
├─ Dev 1: Feature auth → branch: feat/sso
├─ Dev 2: Feature payments → branch: feat/recurring
├─ Dev 3: Feature AI → branch: feat/claude-integration
└─ Merge hell cada 2 días

TRIGGER: ✅ Separar en repos independientes
```

#### Trigger 3: Compliance o Regulación

```
SÍNTOMA:
PCI DSS Level 1, SOC 2, ISO 27001 requieren separación.

EJEMPLO:
Payment Service debe estar separado físicamente
por compliance PCI DSS Level 1.

TRIGGER: ✅ Extraer Payment Service obligatorio
```

#### Trigger 4: Costo de Deploy Alto

```
SÍNTOMA:
Deploy de monolito toma >10 min, afecta todo el sistema.

EJEMPLO:
├─ Build time: 8 min
├─ Test time: 5 min
├─ Deploy time: 3 min
└─ Total: 16 min por deploy

Deploy de microservicio:
└─ 2-3 min solo el servicio afectado

TRIGGER: ✅ Separar módulos que cambian frecuentemente
```

#### Trigger 5: Technology Mismatch

```
SÍNTOMA:
Un módulo necesita stack diferente por performance.

EJEMPLO:
PDF Generation en Node.js: 2-3s
PDF Generation en Go: 200-300ms

O Machine Learning requiere Python.

TRIGGER: ✅ Extraer módulo a servicio con stack óptimo
```

---

## Migration Strategy: Strangler Pattern

### Fase 1: Identificar Candidato (1 semana)

```
Criterios de priorización:
1. Alto overhead de recursos (CPU, RAM, costo)
2. Escala independiente del resto
3. Bounded context claro
4. Ya tiene interfaz bien definida
5. Bajo acoplamiento con otros módulos

Scoring:
┌──────────────┬───────┬────────┬──────────┬───────┬───────┐
│ Módulo       │ CPU   │ Escala │ Context  │ API   │ Score │
├──────────────┼───────┼────────┼──────────┼───────┼───────┤
│ AI Gen       │ ⭐⭐⭐⭐⭐ │ ⭐⭐⭐⭐⭐  │ ⭐⭐⭐⭐⭐   │ ⭐⭐⭐⭐⭐ │ 20/20 │
│ PDF          │ ⭐⭐⭐⭐  │ ⭐⭐⭐⭐   │ ⭐⭐⭐⭐⭐   │ ⭐⭐⭐⭐  │ 17/20 │
│ Payment      │ ⭐⭐   │ ⭐⭐⭐    │ ⭐⭐⭐⭐    │ ⭐⭐⭐⭐  │ 13/20 │
│ Auth         │ ⭐    │ ⭐⭐     │ ⭐⭐      │ ⭐⭐⭐   │  8/20 │
│ Templates    │ ⭐    │ ⭐      │ ⭐⭐⭐     │ ⭐⭐⭐   │  8/20 │
└──────────────┴───────┴────────┴──────────┴───────┴───────┘

Resultado: AI Generation = candidato #1
```

### Fase 2: Preparar Interfaz (1-2 semanas)

```typescript
// ANTES: Implementación directa
// src/modules/ai-generation/service.ts
export class AIGenerationService {
  async generate(prompt: string): Promise<string> {
    const response = await this.openai.chat.completions.create({...});
    return response.choices[0].message.content;
  }
}

// DESPUÉS: Interfaz + implementaciones
// src/modules/ai-generation/interface.ts
export interface AIGenerationService {
  generate(prompt: string): Promise<string>;
  estimateCost(prompt: string): Promise<number>;
}

// src/modules/ai-generation/local.service.ts
export class LocalAIService implements AIGenerationService {
  // Implementación actual (OpenAI direct)
}

// src/modules/ai-generation/remote.service.ts
export class RemoteAIService implements AIGenerationService {
  constructor(private httpClient: HttpClient) {}
  
  async generate(prompt: string): Promise<string> {
    // Llamada a servicio externo
    const response = await this.httpClient.post(
      'https://ai-service.legaia.com/api/generate',
      { prompt }
    );
    return response.data.content;
  }
}

// src/modules/ai-generation/index.ts
import { featureFlags } from '@/lib/feature-flags';

export function createAIService(): AIGenerationService {
  if (featureFlags.useRemoteAI) {
    return new RemoteAIService(httpClient);
  }
  return new LocalAIService();
}
```

### Fase 3: Construir Microservicio (2-4 semanas)

```
Nuevo repo: legaia-ai-service

Estructura:
legaia-ai-service/
├── src/
│   ├── api/
│   │   └── generate.ts      # POST /api/generate
│   ├── services/
│   │   ├── openai.service.ts
│   │   ├── claude.service.ts
│   │   └── fallback.service.ts
│   ├── models/
│   │   └── prompt.model.ts
│   └── index.ts
├── tests/
├── Dockerfile
├── k8s/                     # Si usamos K8s
│   ├── deployment.yaml
│   └── service.yaml
└── package.json

Deploy:
├─ Opción 1: Vercel (serverless)
├─ Opción 2: Railway (PaaS)
├─ Opción 3: AWS ECS (containers)
└─ Opción 4: GKE (Kubernetes)
```

### Fase 4: Feature Flag + Canary (1 semana)

```typescript
// src/lib/feature-flags.ts
export const featureFlags = {
  aiServiceRollout: {
    enabled: true,
    percentage: 10, // Empieza con 10%
    userWhitelist: [
      'anderson@legaia.digital',  // Internal testing
    ],
  },
};

// src/modules/ai-generation/index.ts
export function createAIService(userId: string): AIGenerationService {
  const rollout = featureFlags.aiServiceRollout;
  
  if (!rollout.enabled) {
    return new LocalAIService(); // Fallback siempre
  }
  
  // Whitelist
  if (rollout.userWhitelist.includes(userId)) {
    return new RemoteAIService();
  }
  
  // Canary rollout
  const hash = hashUserId(userId);
  if (hash % 100 < rollout.percentage) {
    return new RemoteAIService();
  }
  
  return new LocalAIService();
}
```

### Fase 5: Incrementar Gradualmente (2-4 semanas)

```
Semana 1: 10% tráfico → monitorear
Semana 2: 25% tráfico → comparar métricas
Semana 3: 50% tráfico → validar costos
Semana 4: 100% tráfico → full migration

Métricas a monitorear:
├─ Latencia P95, P99
├─ Error rate
├─ Costo por request
├─ Throughput
└─ User satisfaction (NPS)

Criterios de éxito:
✅ Latencia similar o mejor
✅ Error rate < 0.1%
✅ Costo < o = monolito
✅ Zero incidents críticos
```

### Fase 6: Cleanup Monolito (1 semana)

```
Cuando 100% tráfico en microservicio:

1. Remover código viejo:
   - src/modules/ai-generation/local.service.ts ❌
   
2. Simplificar:
   - Solo mantener RemoteAIService ✅
   
3. Actualizar tests:
   - Mock remote service en tests
   
4. Documentar:
   - Actualizar ADRs
   - Actualizar arquitectura diagrams
```

---

## Priorización de Servicios

### Tier 1: Candidatos Ideales (Extraer primero)

#### 1. AI Generation Service ⭐⭐⭐⭐⭐

```
Razones:
✅ Alto uso CPU/memoria (GPT-4 processing)
✅ Escala 10x independiente del resto
✅ Bounded context perfecto
✅ API ya bien definida
✅ Costo significativo (OpenAI API)

Trigger específico:
└─ >1000 generaciones/día
└─ Costo OpenAI >$500/mes

Beneficios:
├─ Escalar solo AI cuando necesario
├─ Probar modelos alternativos (Claude, Llama)
├─ A/B testing de prompts
├─ Caching agresivo
└─ Ahorro: ~40% costo total
```

#### 2. PDF Service ⭐⭐⭐⭐

```
Razones:
✅ Proceso CPU-intensive
✅ Puede usar stack optimizado (Go, Python)
✅ Bounded context claro
✅ No depende de otros módulos

Trigger específico:
└─ >500 PDFs/día
└─ Latencia >3s consistente

Beneficios:
├─ Workers dedicados con más CPU
├─ Queue system para batch
├─ Reescribir en Go (10x faster)
└─ Ahorro: ~30% tiempo generación
```

### Tier 2: Considerar Después

#### 3. Payment Service ⭐⭐⭐

```
Razones:
⚠️ Compliance puede requerir separación (PCI DSS Level 1)
⚠️ Alto overhead de seguridad
✅ Bounded context claro
✅ Ya tiene interfaz definida (Stripe)

Trigger específico:
└─ PCI DSS Level 1 requerido
└─ >$100K/mes en pagos

Beneficios:
├─ Compliance simplificado
├─ Security audit independiente
├─ Team dedicado a pagos
└─ Zero impacto en otros servicios
```

#### 4. Notification Service ⭐⭐

```
Razones:
✅ Async por naturaleza
✅ No critical path
⚠️ Bajo overhead actualmente

Trigger específico:
└─ >10K emails/día
└─ SMS, push notifications agregados

Beneficios:
├─ Queue-based (SQS, RabbitMQ)
├─ Retry logic independiente
├─ No bloquea otros servicios
└─ Scaling independiente
```

### Tier 3: Mantener en Monolito

#### ❌ Auth Module

```
Razones:
❌ Bajo overhead (login infrecuente)
❌ Critical para seguridad (mejor en monolito)
❌ Shared state con toda la app
❌ Latencia crítica (mejor in-process)

Mantener en monolito hasta:
└─ Compliance extremo (ISO 27001 Level 3)
```

#### ❌ Template Module

```
Razones:
❌ Básicamente CRUD simple
❌ Casi nunca cambia
❌ Bajo tráfico
❌ No justifica separación

Mantener en monolito indefinidamente.
```

#### ❌ User Profile / Settings

```
Razones:
❌ CRUD simple
❌ Bajo tráfico
❌ Shared con Auth

Mantener en monolito indefinidamente.
```

---

## Arquitectura Target (Año 3-5)

```
┌─────────────────────────────────────────────────────────┐
│                     FRONTEND (Next.js)                   │
│                  Hosted on Vercel/CloudFlare             │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                   API GATEWAY / BFF                      │
│              (Next.js API Routes + Auth)                 │
└──────┬──────────────┬──────────────┬───────────────┬────┘
       │              │              │               │
       ▼              ▼              ▼               ▼
┌────────────┐ ┌────────────┐ ┌───────────┐ ┌──────────────┐
│ MONOLITO   │ │ AI Service │ │ PDF Svc   │ │ Payment Svc  │
│            │ │            │ │           │ │              │
│ • Auth     │ │ • OpenAI   │ │ • Go app  │ │ • Stripe     │
│ • Templates│ │ • Claude   │ │ • Puppeteer│ │ • PCI comp  │
│ • Users    │ │ • Prompts  │ │ • Workers │ │ • Webhooks   │
│ • Settings │ │ • Fallback │ │           │ │              │
└────────────┘ └────────────┘ └───────────┘ └──────────────┘
       │              │              │               │
       └──────────────┴──────────────┴───────────────┘
                         │
                         ▼
            ┌────────────────────────┐
            │  SHARED DATABASES      │
            │  • PostgreSQL (Supabase)│
            │  • Redis (Cache)       │
            │  • S3 (Storage)        │
            └────────────────────────┘
```

**Características:**
- 70% código en monolito
- 3 microservicios estratégicos
- BFF (Backend-for-Frontend) en Next.js
- Databases compartidas (OK para escala media)
- API Gateway simple (no Istio, no Kong)

---

## Anti-Patterns to Avoid

### ❌ Big Bang Migration

```
MAL:
"Vamos a migrar TODO a microservicios en 1 mes"

Resultado:
├─ 6 meses de downtime
├─ Budget overrun 3x
├─ Team burnout
└─ Rollback imposible

BUENO:
"Vamos a extraer AI Service en 6 semanas,
medir, aprender, decidir siguiente paso"
```

### ❌ Microservices for Everything

```
MAL:
├─ auth-service
├─ user-service
├─ template-service
├─ document-service
├─ ai-service
├─ pdf-service
├─ payment-service
├─ notification-service
├─ analytics-service
├─ settings-service
└─ logging-service (11 servicios)

Overhead:
├─ 11 repos
├─ 11 CI/CD pipelines
├─ 11 deployments
├─ Service mesh necesario
└─ 2-3 SREs full-time

BUENO:
├─ monolito (auth, templates, users, settings)
├─ ai-service
├─ pdf-service
└─ payment-service (4 componentes)
```

### ❌ Shared Database per Service

```
MAL:
Cada servicio su propia DB
→ Joins imposibles
→ Data duplication
→ Consistency nightmare

BUENO (para escala media):
Database compartida hasta >10K users/día
Luego Database per Domain (3-4 DBs max)
```

### ❌ Distributed Monolith

```
SÍNTOMA:
├─ Microservicios con acoplamiento fuerte
├─ Servicio A llama B llama C llama D
├─ Cambiar feature requiere 5 deploys
└─ Peor que monolito

PREVENCIÓN:
├─ Bounded contexts claros
├─ Async communication (events)
├─ Evitar sync calls en cadena
└─ Cada servicio independiente
```

---

## Cost Analysis

### Monolito (Actual)

```
Vercel Pro: $20/mes
Supabase Pro: $25/mes
OpenAI: $300/mes
Stripe fees: 2.9% + $0.30
─────────────────────────
Total: ~$350/mes + transaction fees
```

### Híbrido (Futuro - 3 microservicios)

```
Frontend (Vercel): $20/mes
API Gateway (Vercel): included
Monolito (Railway): $25/mes

AI Service (Railway/GCP):
├─ Base: $50/mes
├─ Auto-scale: $100-300/mes peak
└─ Avg: $150/mes

PDF Service (Railway): $25/mes

Payment Service (Railway): $25/mes

Supabase Pro: $25/mes
Redis (Upstash): $10/mes
OpenAI: $300/mes
Monitoring (Sentry): $26/mes
─────────────────────────
Total: ~$656/mes + transaction fees

Incremento: +$300/mes (+85%)
```

### Justificación

```
Costo extra: $300/mes
Ahorro en escala: $500/mes (no sobre-provisionar monolito)
Net: +$200/mes ahorro

Además:
✅ Deploy velocity 2x
✅ Team productivity +30%
✅ Incident resolution 50% faster
```

**Solo migrar si ahorro > costo incremental**

---

## Success Criteria

### Migración Exitosa =

```
✅ Latencia: < o = monolito
✅ Error rate: <0.1%
✅ Costo: < o = monolito a misma escala
✅ Deploy frequency: >=monolito
✅ MTTR: <=monolito
✅ Team velocity: >=monolito
✅ Zero data loss
✅ Zero downtime >5 min
```

### Señales de Alerta

```
❌ Latencia +50%
❌ Error rate >1%
❌ Costo +100% sin benefit claro
❌ Debugging time 2x
❌ Team velocity -20%
❌ Incidents frequency 2x

→ Considerar rollback
→ Reevaluar decisión
```

---

## Timeline Projection

### Optimista (Todo sale bien)

```
Año 1 (2026):
└─ Monolito 100%

Año 2 (2027):
├─ Monolito 90%
└─ AI Service extracted (if triggers met)

Año 3 (2028):
├─ Monolito 80%
├─ AI Service
└─ PDF Service

Año 4 (2029):
├─ Monolito 70%
├─ AI Service
├─ PDF Service
└─ Payment Service (if compliance)

Año 5+ (2030):
└─ Stable hybrid architecture
```

### Realista (Algunos retrasos)

```
Año 1-2:
└─ Monolito 100%

Año 3:
├─ Monolito 95%
└─ AI Service (pilot)

Año 4:
├─ Monolito 85%
├─ AI Service
└─ PDF Service

Año 5+:
└─ 3-4 servicios max
```

---

## Related Decisions

- **ADR-001:** Monolith Modular Architecture
- **ADR-002:** Serverless Deployment
- **ADR-004:** Module Dependency Rules
- **ADR-007:** Event-Driven Architecture (future)

---

## References

- [Strangler Fig Pattern - Martin Fowler](https://martinfowler.com/bliki/StranglerFigApplication.html)
- [Monolith to Microservices - Sam Newman](https://www.nginx.com/blog/microservices-reference-architecture-nginx-deployment/)
- [When to use microservices - Martin Fowler](https://martinfowler.com/articles/microservice-trade-offs.html)
- [Shopify's Modular Monolith Journey](https://shopify.engineering/deconstructing-monolith-designing-software-maximizes-developer-productivity)

---

## Changelog

- **2026-01-27:** Initial strategy - Anderson Aguiar
- **Status:** Deferred until triggers met
- **Next Review:** 2027 Q1 or when 1000 daily active users

---

**Aprobado por:**  
Anderson Aguiar (CTO) - 27/01/2026

**Nota:** Este ADR es una guía para el futuro. La decisión actual es **NO MIGRAR**. Revisar cuando se cumplan triggers específicos.