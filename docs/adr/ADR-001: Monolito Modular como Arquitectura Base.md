# ADR-001: Monolito Modular como Arquitectura Base

**Fecha:** 27 de Enero, 2026  
**Status:** ✅ **Accepted**  
**Decisor:** Anderson Aguiar (CTO)  
**Stakeholders:** Katherine Aguiar (CLO), María Juliana Grajales (CLO)

---

## Context

LegaIA está en fase de MVP con los siguientes constraints:

### Recursos Disponibles
- **Equipo técnico:** 1 desarrollador full-time (Anderson)
- **Timeline:** 8 semanas para MVP funcional
- **Presupuesto inicial:** Bootstrap ($0-50/mes primeros 3 meses)
- **Objetivo Year 1:** 280 usuarios, $185K revenue

### Requerimientos Técnicos
- Generación de documentos legales con IA (GPT-4)
- Sistema de autenticación y autorización
- Procesamiento de pagos (Stripe)
- Conversión HTML → PDF
- Almacenamiento de documentos
- Sistema de suscripciones

### Alternativas Consideradas
1. **Monolito tradicional** (ej: Ruby on Rails, Django)
2. **Monolito modular serverless** (Next.js + Vercel)
3. **Microservicios** (5+ servicios separados)
4. **JAMstack puro** (Frontend + BaaS)

---

## Decision

**Adoptamos arquitectura de Monolito Modular con deployment serverless** utilizando:
- Next.js 14 (App Router) como framework principal
- Organización en módulos de dominio claramente separados
- Interfaces bien definidas entre módulos
- Deployment serverless en Vercel
- Backend-as-a-Service (Supabase) para base de datos, auth y storage

### Definición de Monolito Modular

```
┌─────────────────────────────────────────────────┐
│         LEGAIA - MONOLITO MODULAR               │
│                                                 │
│  Single Codebase, Multiple Modules              │
│                                                 │
│  ┌──────────────┐  ┌──────────────┐            │
│  │ Auth Module  │  │ Document     │            │
│  │              │  │ Module       │            │
│  │ - Service    │  │              │            │
│  │ - Repository │  │ - Service    │            │
│  │ - Types      │  │ - Repository │            │
│  └──────────────┘  │ - Types      │            │
│                     └──────────────┘            │
│  ┌──────────────┐  ┌──────────────┐            │
│  │ AI Gen       │  │ Payment      │            │
│  │ Module       │  │ Module       │            │
│  │              │  │              │            │
│  │ - Service    │  │ - Service    │            │
│  │ - Client     │  │ - Repository │            │
│  │ - Types      │  │ - Types      │            │
│  └──────────────┘  └──────────────┘            │
│                                                 │
│  Características:                               │
│  • Bajo acoplamiento entre módulos             │
│  • Alta cohesión dentro de módulos             │
│  • Interfaces explícitas                       │
│  • Testeable en aislamiento                    │
│  • Preparado para extracción futura            │
└─────────────────────────────────────────────────┘
```

---

## Consequences

### ✅ Positive Consequences

#### 1. **Velocidad de Desarrollo**
- **Impacto:** Alto
- Un solo repositorio = menos overhead de coordinación
- No necesidad de service discovery, API gateways, o message queues
- Desarrollo end-to-end de features sin cambiar contextos
- **Estimación:** 8 semanas para MVP vs 16-24 con microservicios

#### 2. **Bajo Costo Operacional**
- **Impacto:** Crítico para bootstrap
```
Costo Mensual (primeros 3 meses):
├─ Vercel: $0 (free tier)
├─ Supabase: $0 (free tier 500MB)
├─ OpenAI: $20-50 (pay-as-you-go)
├─ Domain: $1
└─ TOTAL: $21-51/mes

vs Microservicios:
└─ TOTAL: $600-900/mes (K8s, monitoring, etc)
```

#### 3. **Simplicidad de Deploy**
- **Impacto:** Alto
- Deploy: `git push` → Vercel auto-deploy (2 min)
- Rollback: `vercel rollback` (30 seg)
- Zero-downtime deployments automáticos
- No necesidad de orchestration (K8s, Docker Swarm, etc)

#### 4. **Debugging Simplificado**
- **Impacto:** Alto
- Stack traces completos en un solo proceso
- Logs centralizados nativamente
- No necesidad de distributed tracing
- Profiling directo con herramientas estándar

#### 5. **Transacciones ACID**
- **Impacto:** Alto (crítico para pagos)
- Transacciones de base de datos nativas
- Consistencia fuerte garantizada
- No necesidad de Saga pattern o compensating transactions
```typescript
// Transacción atómica simple
await db.transaction(async (trx) => {
  await trx.insert(payments).values({...});
  await trx.update(documents).set({paid: true});
  await trx.update(subscriptions).set({used: used + 1});
});
// Todo o nada, consistencia garantizada
```

#### 6. **Testing Simplificado**
- **Impacto:** Medio
- Unit tests estándar (Jest)
- E2E tests con Playwright en un solo servicio
- No necesidad de contract testing entre servicios
- Cobertura más fácil de medir y mantener

#### 7. **Suficiente para Escala Inicial**
- **Impacto:** Alto
- Serverless auto-scaling hasta 10K+ requests/min
- Adecuado para 280-1000 usuarios (objetivo Year 1)
- Headroom para 3-5 años de crecimiento orgánico

### ⚠️ Negative Consequences

#### 1. **Escalabilidad Granular Limitada**
- **Impacto:** Bajo (corto plazo), Medio (largo plazo)
- Si AI Generation necesita 10x recursos, todo el monolito escala
- Costo de scaling menos óptimo que microservicios selectivos
- **Mitigación:** Serverless auto-scaling por route en Vercel reduce este problema

#### 2. **Deploy Acoplado**
- **Impacto:** Medio
- Un bug en módulo Auth requiere deploy completo
- Riesgo de regresión en módulos no relacionados
- **Mitigación:** 
  - Feature flags para activar/desactivar módulos
  - Testing exhaustivo pre-deploy
  - Rollback rápido (<30 seg)

#### 3. **Dificultad con Múltiples Equipos**
- **Impacto:** Bajo (actualmente irrelevante)
- Un solo repo puede generar merge conflicts con 5+ devs
- Coordinación necesaria para cambios grandes
- **Mitigación:** 
  - Actualmente 1 dev, no aplica
  - Cuando crezca equipo (3+ devs), modularización permite extracción

#### 4. **Technology Lock-in por Módulo**
- **Impacto:** Bajo
- Todo el monolito usa TypeScript/Node.js
- Difícil usar Python para ML o Go para performance crítico
- **Mitigación:** 
  - Si necesario, servicios auxiliares separados (ej: worker de ML)
  - GPT-4 API = no necesitamos ML propio por ahora

#### 5. **Riesgo de Acoplamiento Accidental**
- **Impacto:** Medio
- Fácil crear dependencias circulares entre módulos
- Compartir código puede llevar a tight coupling
- **Mitigación:**
  - Reglas de arquitectura estrictas (ver ADR-004)
  - Code reviews enfocados en boundaries
  - Dependency graphs automáticos (dependency-cruiser)

---

## Alternatives Considered

### Alternative 1: Monolito Tradicional (Django/Rails)

**Pros:**
- Framework maduro con muchas baterías incluidas
- ORM robusto
- Admin panel out-of-the-box

**Cons:**
- Deployment no serverless = costo base alto ($50-100/mes)
- Escala vertical limitada sin DevOps
- No auto-scaling
- Time to market similar pero menos moderno

**Razón de rechazo:** Costo operacional alto, no serverless

---

### Alternative 2: Microservicios desde Día 1

**Arquitectura hipotética:**
```
Services:
├─ auth-service (Node.js)
├─ template-service (Node.js)
├─ ai-generation-service (Python/Node.js)
├─ pdf-service (Python)
├─ payment-service (Node.js)
├─ storage-service (Node.js)
└─ notification-service (Node.js)

Infrastructure:
├─ API Gateway (Kong/AWS API Gateway)
├─ Service Discovery (Consul/Eureka)
├─ Message Queue (RabbitMQ/SQS)
├─ Kubernetes (GKE/EKS)
├─ Istio Service Mesh
└─ Distributed Tracing (Jaeger)
```

**Pros:**
- Escalabilidad granular perfecta
- Equipos pueden trabajar independientemente
- Stack heterogéneo (Node.js + Python + Go)
- Deploy independiente por servicio

**Cons:**
- **Complejidad:** 10x mayor que monolito
- **Costo:** $600-900/mes desde día 1
- **Time to market:** 16-24 semanas (3x más)
- **Overhead DevOps:** Requiere 1 SRE dedicado
- **Testing:** Contract testing, chaos engineering necesarios
- **Debugging:** Distributed tracing obligatorio
- **Transacciones:** Saga pattern, eventual consistency

**Análisis de costo-beneficio:**
```
BENEFICIOS:
└─ Escalabilidad granular
   └─ Solo relevante >10K usuarios/día
   └─ LegaIA: 280 usuarios Year 1 = irrelevante

COSTOS:
├─ $600/mes extra (vs $0 con monolito)
├─ 16 semanas extra de desarrollo
├─ Complejidad que ralentiza features
└─ Requiere equipo más grande

CONCLUSIÓN: Costo >>> Beneficio para MVP
```

**Razón de rechazo:** Overengineering, costo injustificado, time-to-market inaceptable

---

### Alternative 3: JAMstack Puro (Frontend + BaaS únicamente)

**Arquitectura hipotética:**
```
Frontend:
└─ Next.js static export

Backend:
├─ Firebase Functions
├─ Supabase Edge Functions
└─ Clerk Auth
```

**Pros:**
- Ultra simple
- Costo bajísimo
- CDN global automático

**Cons:**
- Vendor lock-in extremo
- Edge Functions limitadas (tiempo ejecución, memoria)
- Difícil integración IA compleja
- Migración futura costosa

**Razón de rechazo:** Limitaciones de Edge Functions para IA, vendor lock-in

---

## Comparison Matrix

| Criterio | Monolito Modular | Microservicios | JAMstack Puro |
|----------|------------------|----------------|---------------|
| Time to Market | ✅ 8 sem | ❌ 20 sem | ✅ 6 sem |
| Costo Inicial | ✅ $21/mes | ❌ $600/mes | ✅ $10/mes |
| Escalabilidad | ✅ 10K users | ✅ 1M users | ⚠️ 5K users |
| Complejidad | ✅ Baja | ❌ Alta | ✅ Muy baja |
| Flexibilidad IA | ✅ Alta | ✅ Alta | ⚠️ Limitada |
| Vendor Lock-in | ⚠️ Medio | ✅ Bajo | ❌ Alto |
| Team Size | ✅ 1 dev | ❌ 5+ devs | ✅ 1 dev |
| Testing | ✅ Simple | ❌ Complejo | ✅ Simple |
| Evolvabilidad | ✅ Alta | ✅ Alta | ⚠️ Media |

**Ganador para MVP:** Monolito Modular ✅

---

## Implementation Guidelines

### Principios de Modularización

#### 1. **Separation of Concerns**
```typescript
// ✅ CORRECTO: Módulo autocontenido
src/modules/documents/
├── service.ts          // Lógica de negocio
├── repository.ts       // Acceso a datos
├── types.ts            // Tipos/interfaces
└── index.ts            // Public API del módulo

// service.ts
export class DocumentService {
  constructor(
    private repo: DocumentRepository,
    private aiService: AIGenerationService // Inyección de dependencia
  ) {}
  
  async generate(templateId: string, answers: Answers): Promise<Document> {
    // Lógica aquí
  }
}
```

#### 2. **Interface-Driven Design**
```typescript
// ✅ CORRECTO: Dependencia de interfaz, no implementación
export interface AIGenerationService {
  generate(prompt: string): Promise<string>;
  estimateCost(prompt: string): Promise<number>;
}

export class OpenAIService implements AIGenerationService {
  // Implementación específica
}

// Mañana puedes cambiar a Claude, local model, o servicio externo
export class ClaudeService implements AIGenerationService {
  // Diferente implementación, misma interfaz
}
```

#### 3. **Dependency Rules**
```
Regla: Los módulos solo pueden depender de módulos de igual o menor nivel

Nivel 1 (Core):
├─ types/
└─ utils/

Nivel 2 (Infrastructure):
├─ database/
├─ external-apis/ (OpenAI, Stripe)
└─ storage/

Nivel 3 (Domain):
├─ auth/
├─ templates/
├─ documents/
├─ ai-generation/
└─ payments/

Nivel 4 (Application):
└─ app/ (Next.js routes)

✅ Nivel 4 puede usar Nivel 1, 2, 3
✅ Nivel 3 puede usar Nivel 1, 2
❌ Nivel 2 NO puede usar Nivel 3
❌ Nivel 1 NO puede usar nada externo
```

#### 4. **Testing Strategy**
```typescript
// Unit tests por módulo
src/modules/documents/__tests__/
├── service.test.ts
├── repository.test.ts
└── integration.test.ts

// E2E tests por flujo de usuario
tests/e2e/
├── auth-flow.spec.ts
├── document-generation-flow.spec.ts
└── payment-flow.spec.ts
```

---

## Migration Strategy (Future)

Cuando llegue el momento de migrar a microservicios (>1000 users, >$500K revenue):

### Fase 1: Identificar Candidatos
```
Criterios para extracción:
✅ AI Generation Service
   - Alto uso de CPU/memoria
   - Escalabilidad independiente crítica
   - Ya tiene interfaz bien definida
   
✅ PDF Service
   - Proceso pesado
   - Candidato para workers separados
   
⚠️ Payment Service
   - Compliance puede requerir separación
   - Pero bajo volumen inicial
   
❌ Auth
   - Bajo overhead
   - Mantener en monolito
```

### Fase 2: Strangler Pattern
```
1. Crear servicio externo (ej: ai-generation-service)
2. Implementar feature flag
3. Rutear X% tráfico al nuevo servicio
4. Monitorear y comparar
5. Aumentar gradualmente a 100%
6. Eliminar código viejo del monolito
```

### Fase 3: Anti-Corruption Layer
```typescript
// En monolito
export class AIGenerationService {
  async generate(prompt: string): Promise<string> {
    if (featureFlags.useExternalAIService) {
      // Llamar servicio externo
      return await httpClient.post('https://ai-service.legaia.com/generate', {
        prompt
      });
    } else {
      // Implementación local (fallback)
      return await this.openai.generate(prompt);
    }
  }
}
```

---

## Metrics and Success Criteria

### Métricas de Éxito del Monolito Modular

#### Performance
- ✅ **Latencia P95:** <500ms (generación documentos: <5s)
- ✅ **Availability:** >99.5%
- ✅ **Error rate:** <1%

#### Escalabilidad
- ✅ **Concurrent users:** 100+
- ✅ **Requests/min:** 1000+
- ✅ **Auto-scaling:** 0-10 instancias automático

#### Costo
- ✅ **Mes 1-3:** <$100/mes
- ✅ **Mes 4-12:** <$1000/mes
- ✅ **Cost per user:** <$3/mes

#### Desarrollo
- ✅ **Time to MVP:** 8 semanas
- ✅ **Deploy frequency:** Daily
- ✅ **MTTR:** <15 min
- ✅ **Test coverage:** >80%

### Triggers para Reevaluación

Revisar decisión si:
- ❌ Latencia P95 > 1s consistentemente
- ❌ Costo > $5/usuario/mes
- ❌ Deploy time > 10 min
- ❌ Más de 3 devs bloqueados por merge conflicts
- ❌ Un módulo necesita escalar 10x independiente

---

## Related Decisions

- **ADR-002:** Serverless Deployment Strategy (Vercel)
- **ADR-003:** Migration Strategy to Microservices
- **ADR-004:** Module Dependency Rules
- **ADR-005:** Database Schema Design (Single Database)

---

## References

- [Monolith First by Martin Fowler](https://martinfowler.com/bliki/MonolithFirst.html)
- [Modular Monoliths by Simon Brown](https://www.youtube.com/watch?v=5OjqD-ow8GE)
- [Don't start with microservices by Stefan Tilkov](https://martinfowler.com/articles/dont-start-monolith.html)
- [Shopify's Modular Monolith](https://shopify.engineering/shopify-monolith)
- [The Majestic Monolith by DHH](https://m.signalvnoise.com/the-majestic-monolith/)

---

## Changelog

- **2026-01-27:** Initial decision - Anderson Aguiar
- **Future:** To be reviewed at 1000 users milestone

---

**Aprobado por:**  
Anderson Aguiar (CTO) - 27/01/2026

**Revisado por:**  
Katherine Aguiar (CLO) - Done  
María Juliana Grajales (CLO) - Done