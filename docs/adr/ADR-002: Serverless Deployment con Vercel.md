# ADR-002: Serverless Deployment con Vercel

**Fecha:** 27 de Enero, 2026  
**Status:** ✅ **Accepted**  
**Decisor:** Anderson Aguiar (CTO)  
**Stakeholders:** Katherine Aguiar (CLO), María Juliana Grajales (CLO)

---

## Context

LegaIA necesita una estrategia de deployment que sea:
- **Económica:** Presupuesto $0-50/mes en fase MVP
- **Rápida:** Deploy en minutos, no horas
- **Escalable:** Auto-scaling sin intervención manual
- **Simple:** 1 desarrollador debe poder manejar toda la infraestructura
- **Confiable:** >99.5% uptime

### Traffic Patterns Esperados

```
MVP (Primeros 3 meses):
├─ 5-50 usuarios beta
├─ ~200-500 requests/día
├─ Picos: 10-20 requests/min
└─ Patrón: Irregular, pruebas

Growth (Mes 4-12):
├─ 50-280 usuarios
├─ ~2000-5000 requests/día
├─ Picos: 50-100 requests/min
└─ Patrón: Horario laboral (8am-6pm)

Características del tráfico:
✓ B2B = predecible (horario laboral)
✓ Colombia = 1 timezone
✓ Estacionalidad baja
✗ No necesita global CDN inicialmente
```

### Alternativas Consideradas

1. **Vercel (Serverless)**
2. **AWS (EC2 + Load Balancer)**
3. **Railway / Render (PaaS)**
4. **Self-hosted (VPS)**
5. **Kubernetes (GKE/EKS)**

---

## Decision

**Adoptamos Vercel como plataforma de deployment** con:
- Next.js 14 App Router deployment automático
- Serverless Functions para API routes
- Edge Network global
- Preview deployments para cada PR
- Zero-config auto-scaling

### Arquitectura de Deployment

```
┌─────────────────────────────────────────────────────────┐
│                    VERCEL EDGE NETWORK                   │
│                     (Global CDN)                         │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│              VERCEL SERVERLESS FUNCTIONS                 │
│                (AWS Lambda bajo el hood)                 │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ GET /        │  │ POST /api/   │  │ POST /api/   │  │
│  │ (Static)     │  │ documents    │  │ payments     │  │
│  │              │  │ (Lambda)     │  │ (Lambda)     │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  Auto-scaling: 0 → N instances                          │
│  Cold start: ~200ms                                     │
│  Timeout: 10s (Free) / 60s (Pro)                        │
└────────────────────────┬────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────┐
│                  EXTERNAL SERVICES                       │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  SUPABASE    │  │   OPENAI     │  │    STRIPE    │  │
│  │  (Database)  │  │   (GPT-4)    │  │  (Payments)  │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────┘

Flow de Deploy:
1. git push origin main
2. Vercel detecta push
3. Build automático (2-3 min)
4. Deploy a producción (30 seg)
5. URL live: legaia.digital
```

---

## Consequences

### ✅ Positive Consequences

#### 1. **Costo Extremadamente Bajo (Fase MVP)**

**Free Tier incluye:**
```
Vercel Free Tier:
├─ 100 GB bandwidth/mes
├─ Serverless Functions: 100 GB-hours/mes
├─ Edge Functions: 500K invocations/mes
├─ Unlimited deployments
├─ Automatic HTTPS
├─ Preview deployments
└─ COSTO: $0/mes

Estimación uso LegaIA MVP (50 users):
├─ Bandwidth: ~5 GB/mes
├─ Function executions: ~15K/mes
├─ Build minutes: ~100 min/mes
└─ Headroom: 95% disponible ✅
```

**Pro Tier (cuando necesario):**
```
Vercel Pro: $20/mes
├─ 1 TB bandwidth
├─ 1000 GB-hours functions
├─ Edge Functions: Unlimited
├─ 60s timeout (vs 10s free)
├─ Priority support
└─ Advanced analytics

Cuándo upgrade:
└─ >50 GB bandwidth/mes (aprox 200-300 users)
```

#### 2. **Deploy Automático y Rápido**

```bash
# Deploy a producción
git push origin main

# Vercel automáticamente:
✓ Detecta push (5 seg)
✓ Build (2-3 min)
  ├─ npm install
  ├─ npm run build
  └─ Optimización assets
✓ Deploy (30 seg)
✓ Invalidación CDN (10 seg)
✓ Health check (5 seg)
✓ Live en legaia.digital

Total: ~3-4 minutos
```

**vs Alternativas:**
```
AWS EC2 manual:
├─ SSH al servidor: 30s
├─ git pull: 10s
├─ npm install: 2 min
├─ npm run build: 3 min
├─ Restart PM2: 30s
├─ Verificar health: 1 min
└─ Total: ~7 min + manual ❌

Kubernetes:
├─ Build Docker image: 5 min
├─ Push to registry: 2 min
├─ kubectl apply: 1 min
├─ Rolling update: 3 min
├─ Verificar pods: 1 min
└─ Total: ~12 min + manual ❌
```

#### 3. **Preview Deployments (Crítico para QA)**

```
Cada PR en GitHub genera:
├─ URL única: legaia-pr-123.vercel.app
├─ Build aislado
├─ Comentario automático en PR con link
└─ Permite testing antes de merge

Benefits:
✓ Katherine y María Juliana pueden probar features
✓ Zero riesgo en producción
✓ Feedback rápido
✓ No necesidad de staging server
```

#### 4. **Auto-Scaling Sin Configuración**

```
Traffic Pattern:
8am: 0 users    → 0 instances  ($0)
9am: 10 users   → 2 instances  (auto)
10am: 50 users  → 5 instances  (auto)
12pm: 20 users  → 2 instances  (auto)
6pm: 0 users    → 0 instances  ($0)

Features:
✓ Scale to zero (no costo idle)
✓ Scale up en <1s
✓ Maneja spikes automáticamente
✓ No capacity planning necesario
```

**vs Alternativas:**
```
EC2:
❌ Costo 24/7 aunque no hay tráfico
❌ Manual scaling (Auto Scaling Groups)
❌ Capacity planning necesario
❌ Over-provisioning para picos

Kubernetes:
❌ Cluster base: 2-3 nodes mínimo
❌ Configuración compleja (HPA)
❌ Costo base alto
```

#### 5. **Global Edge Network**

```
Vercel CDN locations:
├─ North America: 15+ locations
├─ South America: São Paulo, Santiago
├─ Europe: 20+ locations
├─ Asia: 10+ locations
└─ Total: 70+ locations worldwide

Para LegaIA:
Primary: Colombia
Edge: São Paulo (40ms latency)
Fallback: North Virginia (120ms)

Static assets:
✓ Cached en edge
✓ < 50ms response time global
✓ Imágenes, CSS, JS optimizados automático
```

#### 6. **Zero DevOps**

```
Vercel maneja:
✓ Load balancing
✓ SSL certificates (Let's Encrypt automático)
✓ DDoS protection
✓ Firewall
✓ Monitoring básico
✓ Logs (7 días gratis, 30 días Pro)
✓ Health checks
✓ Rollback (1 click)

Anderson NO necesita:
❌ Configurar Nginx
❌ Configurar SSL
❌ Setup load balancer
❌ Configurar firewall
❌ Setup monitoring
❌ Manage servers
❌ Security patches
```

#### 7. **Integración Perfecta con Next.js**

```
Features automáticas:
✓ App Router support
✓ API Routes → Serverless Functions
✓ Image Optimization
✓ ISR (Incremental Static Regeneration)
✓ Edge Runtime cuando necesario
✓ Middleware
✓ Analytics (Web Vitals)

Zero configuración necesaria.
```

### ⚠️ Negative Consequences

#### 1. **Cold Starts**

```
Problema:
Primera request a function inactiva: ~200-400ms
Requests subsecuentes: ~20-50ms

Mitigación:
✓ Pro tier: Functions más calientes
✓ Pings periódicos si crítico
✓ Edge Functions para latencia crítica
✓ Static generation donde posible

Impacto en LegaIA:
⚠️ Bajo - B2B no requiere latencia ultra-baja
✓ Generación documento: 2-5s (cold start irrelevante)
```

#### 2. **Vendor Lock-in (Moderado)**

```
Dependencias Vercel-específicas:
├─ Vercel Edge Functions
├─ Vercel Image Optimization
├─ Vercel Analytics
└─ Vercel KV (si se usa)

Mitigación:
✓ Core app es Next.js estándar
✓ Puede deployar en Netlify, Railway, AWS
✓ Cambio: ~2-3 días trabajo
✓ No lock-in crítico

Lock-in score: 3/10 (bajo)
```

#### 3. **Límites de Timeout**

```
Free tier: 10 segundos
Pro tier: 60 segundos

Operaciones largas en LegaIA:
✓ Generación documento: 2-5s ✅
✓ PDF conversión: 1-2s ✅
✗ Procesamiento batch: >60s ❌

Solución para batch:
→ Vercel Cron Jobs (Pro)
→ O background worker separado (Supabase Edge Functions)
```

#### 4. **Costo en Escala Alta**

```
Vercel pricing tiers:
├─ Free: $0 (hasta ~100 users)
├─ Pro: $20/mes (hasta ~500 users)
├─ Enterprise: Custom ($2K+/mes)

Break-even con AWS:
~2000-5000 users concurrentes

Para LegaIA:
✓ Year 1-2: Pro tier suficiente ($20/mes)
✓ Year 3+: Evaluar alternativas
```

---

## Alternatives Considered

### Alternative 1: AWS EC2 + Load Balancer

**Arquitectura:**
```
├─ EC2 t3.small (2 vCPU, 2GB RAM): $15/mes
├─ Application Load Balancer: $18/mes
├─ Route 53: $0.50/mes
├─ CloudWatch: $5/mes
├─ Auto Scaling Group (2 instancias min)
└─ TOTAL: $53/mes base + costo instancias escaladas
```

**Pros:**
- Control total
- No vendor lock-in específico
- Pricing predecible

**Cons:**
- **Setup time:** 1-2 semanas
- **DevOps overhead:** Continuo
- **Costo base:** $50/mes aunque no haya tráfico
- **Manual scaling:** Configuración Auto Scaling
- **Security:** Patches, firewall manual
- **SSL:** Configuración Let's Encrypt manual

**Razón de rechazo:** 
- Costo base muy alto para MVP ($50 vs $0)
- Overhead DevOps inaceptable para 1 dev
- Time to market: +2 semanas

---

### Alternative 2: Railway / Render (PaaS)

**Railway:**
```
Pricing:
├─ $5/mes base
├─ + $0.000463/GB-hour RAM
├─ + $0.000231/vCPU-hour
└─ Estimado: $15-25/mes

Features:
✓ Git-based deploys
✓ Free PostgreSQL
✓ Simple dashboard
✓ Good DX
```

**Render:**
```
Pricing:
├─ Web Service: $7/mes
├─ PostgreSQL: $7/mes (500MB)
└─ Total: $14/mes

Features:
✓ Auto-deploys
✓ Free SSL
✓ DDoS protection
```

**Pros:**
- Costo bajo-moderado
- Deploy automático
- Menos lock-in que Vercel

**Cons:**
- **No serverless:** Instancias siempre corriendo
- **No auto-scale to zero:** Costo base inevitable
- **Menos integración Next.js:** Manual config
- **Menos CDN coverage:** Sin edge network
- **Cold starts peores:** Instancias duermen en free tier

**Razón de rechazo:**
- Vercel free tier > Railway/Render pricing
- Mejor integración Next.js en Vercel
- Más CDN locations

---

### Alternative 3: Self-Hosted VPS (DigitalOcean/Linode)

**Arquitectura:**
```
├─ VPS 2GB RAM: $12/mes
├─ Nginx reverse proxy
├─ PM2 process manager
├─ Let's Encrypt SSL
└─ Manual todo
```

**Pros:**
- Costo fijo predecible ($12/mes)
- Control total
- Zero vendor lock-in

**Cons:**
- **Manual everything:**
  - Deploys (SSH, git pull, build, restart)
  - SSL renewal
  - Security patches
  - Monitoring
  - Backups
  - Scaling
- **Single point of failure:** 1 servidor
- **No auto-scaling**
- **No CDN**
- **Overhead time:** 5-10 hrs/mes mantenimiento

**Razón de rechazo:**
- Time overhead inaceptable
- No auto-scaling
- Vercel free tier es mejor

---

### Alternative 4: Kubernetes (GKE/EKS)

**Arquitectura:**
```
├─ GKE Cluster (3 nodes e2-small): $75/mes
├─ Load Balancer: $18/mes
├─ Container Registry: $5/mes
├─ Monitoring (Prometheus/Grafana): $20/mes
└─ TOTAL: $118/mes base
```

**Pros:**
- Máxima flexibilidad
- Industry standard
- Multi-cloud portable

**Cons:**
- **Costo prohibitivo:** $118/mes base
- **Complejidad extrema:** 
  - Helm charts
  - kubectl commands
  - YAML manifests
  - Service meshes
  - Ingress controllers
- **Learning curve:** Semanas/meses
- **Overhead:** SRE tiempo completo

**Razón de rechazo:**
- Overengineering masivo
- Costo 100x Vercel free
- Complejidad injustificada

---

## Comparison Matrix

| Criterio | Vercel | AWS EC2 | Railway | VPS | K8s |
|----------|--------|---------|---------|-----|-----|
| **Costo MVP** | ✅ $0 | ❌ $53 | ⚠️ $15 | ⚠️ $12 | ❌ $118 |
| **Setup Time** | ✅ 0 min | ❌ 2 sem | ⚠️ 2 hrs | ❌ 1 sem | ❌ 1 mes |
| **Deploy Time** | ✅ 3 min | ⚠️ 7 min | ✅ 5 min | ❌ 10 min | ❌ 15 min |
| **Auto-scaling** | ✅ Auto | ⚠️ Config | ❌ No | ❌ No | ⚠️ Complejo |
| **Scale to Zero** | ✅ Sí | ❌ No | ❌ No | ❌ No | ❌ No |
| **DevOps** | ✅ Zero | ❌ Alto | ⚠️ Bajo | ❌ Alto | ❌ Extremo |
| **CDN** | ✅ Global | ⚠️ Manual | ⚠️ Limitado | ❌ No | ⚠️ Manual |
| **Lock-in** | ⚠️ Medio | ✅ Bajo | ⚠️ Medio | ✅ Zero | ✅ Bajo |
| **Preview Env** | ✅ Auto | ❌ Manual | ✅ Auto | ❌ Manual | ❌ Complejo |
| **Monitoring** | ✅ Incluido | ⚠️ CloudWatch | ⚠️ Básico | ❌ Manual | ⚠️ Prometheus |

**Ganador:** Vercel ✅

---

## Implementation Plan

### Phase 1: Initial Setup (Día 1)

```bash
# 1. Instalar Vercel CLI
npm i -g vercel

# 2. Login
vercel login

# 3. Link proyecto
cd legaia
vercel link

# 4. Deploy primera vez
vercel

# 5. Deploy a producción
vercel --prod

# Total time: ~10 minutos
```

### Phase 2: CI/CD Setup (Día 2)

```yaml
# .github/workflows/vercel.yml (opcional, Vercel auto-deploys)
name: Vercel Deployment
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: vercel/action@v1
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
```

### Phase 3: Custom Domain (Día 3)

```bash
# En Vercel dashboard:
1. Settings → Domains
2. Add: legaia.digital
3. Configure DNS (Vercel da instrucciones)
4. Wait propagation (5-30 min)
5. Auto SSL (1 min)

# DNS records (en tu registrar):
Type: A
Name: @
Value: 76.76.21.21 (Vercel IP)

Type: CNAME
Name: www
Value: cname.vercel-dns.com
```

### Phase 4: Environment Variables

```bash
# Via CLI
vercel env add OPENAI_API_KEY production
vercel env add SUPABASE_URL production
vercel env add STRIPE_SECRET_KEY production

# O via Dashboard
# Settings → Environment Variables
# Add OPENAI_API_KEY, etc.
```

---

## Monitoring and Observability

### Vercel Analytics (Incluido)

```
Métricas automáticas:
├─ Web Vitals (LCP, FID, CLS)
├─ Page views
├─ Top pages
├─ Geography
└─ Devices

Pro tier agrega:
├─ Real User Monitoring
├─ Server-side analytics
├─ Audience insights
└─ 90-day retention
```

### External Monitoring (Opcional)

```
Agregar para producción:
├─ Sentry (errores): $0-26/mes
├─ Better Uptime (uptime): $0/mes
└─ LogRocket (session replay): $0-99/mes
```

---

## Cost Projection

### Months 1-3 (MVP, <50 users)

```
Vercel: $0 (Free tier)
├─ Bandwidth: 5 GB / 100 GB available
├─ Build minutes: 100 / 6000 available
├─ Serverless executions: 15K / 100K GB-hours
└─ Headroom: 95% disponible ✅

Total Vercel: $0/mes
```

### Months 4-12 (Growth, 50-280 users)

```
Vercel Pro: $20/mes
├─ Bandwidth: 50 GB / 1 TB available
├─ Serverless executions: 100K / 1000K GB-hours
└─ Headroom: 95% disponible ✅

Cuándo upgrade:
└─ Al superar 100 GB bandwidth O
└─ Necesitar analytics avanzado O
└─ Necesitar 60s timeout

Total Vercel: $20/mes
```

### Year 2+ (Scale, 500-1000 users)

```
Vercel Pro: $20/mes (suficiente)
O
Vercel Enterprise: Custom (si >1TB bandwidth)

Evaluation point:
└─ Si >$200/mes en Vercel, considerar:
    - AWS Amplify
    - CloudFlare Pages
    - Self-hosted con CDN
```

---

## Rollback Strategy

### Instant Rollback

```bash
# Via CLI
vercel rollback

# O via Dashboard
# Deployments → Previous deployment → Promote

# Tiempo: ~30 segundos
# Zero downtime
```

### Preview Deployment Promotion

```bash
# Si preview deployment está OK:
vercel promote <preview-url>

# Ejemplo:
vercel promote legaia-pr-123.vercel.app
```

---

## Security Considerations

### Automatic Security

```
Vercel provides:
✓ DDoS protection (Cloudflare-level)
✓ Rate limiting (default)
✓ SSL/TLS (automatic)
✓ CORS headers (configurable)
✓ Security headers (configurable)
✓ IP blocking (Pro tier)
```

### Custom Security Headers

```typescript
// next.config.js
module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: [
          {
            key: 'X-Frame-Options',
            value: 'DENY',
          },
          {
            key: 'X-Content-Type-Options',
            value: 'nosniff',
          },
          {
            key: 'Referrer-Policy',
            value: 'strict-origin-when-cross-origin',
          },
        ],
      },
    ];
  },
};
```

---

## Success Metrics

### Performance

```
Targets:
├─ Time to First Byte: <200ms
├─ Largest Contentful Paint: <2.5s
├─ Cumulative Layout Shift: <0.1
├─ First Input Delay: <100ms
└─ Deploy time: <5min
```

### Availability

```
Targets:
├─ Uptime: >99.9%
├─ Error rate: <0.5%
├─ P95 latency: <500ms
└─ P99 latency: <1s
```

### Cost Efficiency

```
Targets:
├─ Cost per user: <$0.10/mes
├─ Infrastructure as % revenue: <5%
└─ ROI: Deploy time saved > Vercel cost
```

---

## Migration Path (If Needed)

Si en el futuro necesitamos migrar de Vercel:

### Option 1: Netlify

```
Similarity: 95%
Changes needed:
├─ Update _redirects file
├─ Netlify.toml config
├─ Environment variables
└─ Time: 2-4 hours
```

### Option 2: AWS Amplify

```
Similarity: 80%
Changes needed:
├─ amplify.yml config
├─ Build settings
├─ Environment variables
├─ Custom domain setup
└─ Time: 1 day
```

### Option 3: Self-Hosted

```
Similarity: 60%
Changes needed:
├─ Docker containerization
├─ Nginx configuration
├─ PM2 setup
├─ SSL configuration
├─ CI/CD pipeline
└─ Time: 1 week
```

---

## Related Decisions

- **ADR-001:** Monolith Modular Architecture
- **ADR-003:** Migration Strategy to Microservices
- **ADR-006:** Edge Functions vs Serverless Functions

---

## References

- [Vercel Documentation](https://vercel.com/docs)
- [Next.js Deployment](https://nextjs.org/docs/deployment)
- [Serverless Architectures](https://martinfowler.com/articles/serverless.html)
- [The Cost of Cloud](https://a16z.com/2021/05/27/cost-of-cloud-paradox-market-cap-cloud-lifecycle-scale-growth-repatriation-optimization/)

---

## Changelog

- **2026-01-27:** Initial decision - Anderson Aguiar
- **Future:** Review at 500 users or $200/mes Vercel cost

---

**Aprobado por:**  
Anderson Aguiar (CTO) - 27/01/2026