# Architecture Decision Records (ADRs) - LegaIA

Este directorio contiene todas las decisiones arquitectónicas importantes del proyecto LegaIA, documentadas siguiendo el formato ADR (Architecture Decision Record).

---

## ¿Qué es un ADR?

Un ADR documenta una decisión arquitectónica importante, incluyendo:
- **Context**: Por qué necesitamos tomar la decisión
- **Decision**: Qué decidimos hacer
- **Consequences**: Implicaciones positivas y negativas
- **Alternatives**: Otras opciones consideradas
- **Status**: Accepted, Rejected, Deprecated, Superseded

---

## ADRs Activos

### ADR-001: Monolito Modular como Arquitectura Base
**Status:** ✅ Accepted  
**Fecha:** 27 Enero 2026

**Resumen:**  
LegaIA adopta una arquitectura de **Monolito Modular** con deployment serverless para el MVP y crecimiento inicial (Year 1-3).

**Razones clave:**
- ✅ Time to market: 8 semanas (vs 16-24 con microservicios)
- ✅ Costo: $21-51/mes (vs $600-900/mes)
- ✅ Equipo: 1 dev puede manejarlo
- ✅ Suficiente para 280-1000 usuarios

**Cuándo revisar:**
- Al alcanzar 1000 usuarios activos/día
- Al generar $500K+/año revenue
- Al crecer equipo a 3+ desarrolladores

📄 [Ver ADR completo](./ADR-001-monolith-modular-architecture.md)

---

### ADR-002: Serverless Deployment con Vercel
**Status:** ✅ Accepted  
**Fecha:** 27 Enero 2026

**Resumen:**  
Deployment en **Vercel** con funciones serverless, auto-scaling, y zero-DevOps.

**Razones clave:**
- ✅ Free tier generoso ($0/mes para MVP)
- ✅ Deploy automático (git push → producción en 3 min)
- ✅ Auto-scaling: 0 → N instancias
- ✅ CDN global incluido
- ✅ Preview deployments por PR

**Cuándo revisar:**
- Costo >$200/mes en Vercel
- Necesidad de control total (unlikely)
- Timeouts >60s consistentes

📄 [Ver ADR completo](./ADR-002-serverless-deployment-vercel.md)

---

### ADR-003: Estrategia de Migración a Microservicios
**Status:** ⏸️ Deferred (Para futuro)  
**Fecha:** 27 Enero 2026

**Resumen:**  
Estrategia para **migrar selectivamente** a microservicios cuando se cumplan triggers específicos. No big-bang, sino **Strangler Pattern** gradual.

**Triggers para migrar:**
- ✅ Un módulo necesita escalar 10x independiente
- ✅ Team crece a 5+ devs con merge conflicts frecuentes
- ✅ Compliance requiere separación (PCI DSS Level 1)
- ✅ Revenue >$500K/año que justifica el costo

**Servicios candidatos (en orden):**
1. AI Generation Service ⭐⭐⭐⭐⭐
2. PDF Service ⭐⭐⭐⭐
3. Payment Service ⭐⭐⭐ (si compliance)

**Mantener en monolito:**
- Auth, Templates, User Settings

📄 [Ver ADR completo](./ADR-003-migration-strategy-microservices.md)

---

## ADRs Futuros (Planificados)

### ADR-004: Module Dependency Rules
**Status:** 📝 Draft  
**Razón:** Definir reglas estrictas de dependencias entre módulos

### ADR-005: Database Schema Design
**Status:** 📝 Draft  
**Razón:** Single database vs database-per-module

### ADR-006: Edge Functions vs Serverless Functions
**Status:** 📝 Draft  
**Razón:** Cuándo usar Edge Runtime vs Node.js Runtime

### ADR-007: Event-Driven Architecture
**Status:** 📝 Draft  
**Razón:** Patrón pub/sub para desacoplamiento futuro

### ADR-008: Caching Strategy
**Status:** 📝 Draft  
**Razón:** Redis, CDN, application-level cache

---

## Proceso de ADR

### Cuándo crear un ADR

Crear un ADR cuando:
- ✅ La decisión es difícil de revertir
- ✅ Afecta múltiples partes del sistema
- ✅ Tiene trade-offs significativos
- ✅ El equipo necesita alineación
- ✅ Futuros desarrolladores necesitarán contexto

**NO** crear ADR para:
- ❌ Decisiones triviales (naming conventions)
- ❌ Decisiones fácilmente reversibles
- ❌ Decisiones de implementación local

### Cómo proponer un ADR

1. **Copiar template:**
```bash
cp ADR-TEMPLATE.md ADR-XXX-your-decision.md
```

2. **Llenar secciones:**
- Context: ¿Por qué necesitamos esto?
- Decision: ¿Qué decidimos?
- Consequences: ¿Qué implica?
- Alternatives: ¿Qué otras opciones consideramos?

3. **Crear PR:**
```bash
git checkout -b adr/xxx-your-decision
git add docs/adr/ADR-XXX-your-decision.md
git commit -m "docs: ADR-XXX Your Decision"
git push origin adr/xxx-your-decision
```

4. **Revisión:**
- Anderson (CTO) revisa
- Discusión en equipo si necesario
- Aprobación → Merge

5. **Status update:**
- Initial: 📝 Draft
- After review: ✅ Accepted / ❌ Rejected
- Later: 🔄 Superseded / ⚠️ Deprecated

---

## Revisión de ADRs

### Frecuencia

- **Trimestral:** Review rápida de todos los ADRs activos
- **Anual:** Review profunda + actualización de timeline
- **Ad-hoc:** Cuando triggers específicos se cumplen

### Próximas Reviews

| ADR | Próxima Review | Trigger |
|-----|----------------|---------|
| ADR-001 | Q1 2027 | 1000 usuarios/día |
| ADR-002 | Ongoing | Costo >$200/mes |
| ADR-003 | Q1 2027 | Triggers específicos |

---

## Recursos

### Templates
- [ADR Template](./ADR-TEMPLATE.md)
- [Markdown Guidelines](./MARKDOWN-GUIDE.md)

### Referencias
- [ADR GitHub - Joel Parker Henderson](https://github.com/joelparkerhenderson/architecture-decision-record)
- [Documenting Architecture Decisions - Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [ADR Process - ThoughtWorks](https://www.thoughtworks.com/en-us/radar/techniques/lightweight-architecture-decision-records)

---

## Changelog

- **2026-01-27:** Creación inicial de ADRs 001-003 - Anderson Aguiar
- **Future:** Agregar ADRs 004-008 según necesidad

---

**Mantenido por:** Anderson Aguiar (CTO)  
**Última actualización:** 27 de Enero, 2026