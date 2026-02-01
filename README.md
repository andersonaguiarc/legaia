# LegaIA - Generador Inteligente de Documentos Legales

<div align="center">

![LegaIA Logo](https://via.placeholder.com/150x150?text=LegaIA)

**Democratizando el acceso a documentos legales de calidad en Colombia mediante IA**

[![Next.js](https://img.shields.io/badge/Next.js-14-black?logo=next.js)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue?logo=typescript)](https://www.typescriptlang.org/)
[![Supabase](https://img.shields.io/badge/Supabase-Backend-green?logo=supabase)](https://supabase.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4-412991?logo=openai)](https://openai.com/)
[![License](https://img.shields.io/badge/License-Proprietary-red)]()

[Demo](#) • [Documentación](#) • [Roadmap](#roadmap) • [Equipo](#equipo)

</div>

---

## 🎯 Visión

LegaIA combina inteligencia artificial con expertise legal especializado para generar documentos legales profesionales de manera rápida, precisa y asequible, eliminando la brecha de acceso a servicios legales de calidad para PyMEs en Colombia.

## 💡 Problema

Las pequeñas y medianas empresas en Colombia enfrentan:
- **Costos elevados**: $200-500 USD por documento legal simple
- **Tiempos prolongados**: 3-7 días para documentos básicos
- **Acceso limitado**: No todos pueden pagar un abogado
- **Riesgo de errores**: Usar plantillas genéricas sin asesoría

## ✨ Solución

Plataforma web que genera documentos legales mediante:
1. **Wizard inteligente** de preguntas contextualizadas
2. **IA especializada** (GPT-4) con prompts legales optimizados
3. **Validación experta** de abogados especializados
4. **Entrega inmediata** en PDF descargable

---

## 🛠️ Stack Tecnológico

### Frontend
- **Framework**: Next.js 14 (App Router)
- **Lenguaje**: TypeScript 5.0
- **Estilos**: Tailwind CSS + shadcn/ui
- **Validación**: Zod + React Hook Form
- **State**: React Context + Hooks

### Backend
- **Runtime**: Next.js API Routes (Serverless)
- **Base de datos**: Supabase (PostgreSQL)
- **Autenticación**: Supabase Auth (OAuth + Magic Links)
- **Storage**: Supabase Storage
- **IA**: OpenAI GPT-4 Turbo

### Servicios
- **Pagos**: Stripe (Checkout + Subscriptions)
- **Hosting**: Vercel (Frontend + API)
- **CDN**: Cloudflare
- **Monitoring**: Sentry (próximamente)

### Herramientas
- **Control de versiones**: Git + GitHub
- **Testing**: Jest + Playwright (próximamente)
- **CI/CD**: GitHub Actions (próximamente)
- **Linting**: ESLint + Prettier

---

## 📁 Estructura del Proyecto

```
legaia-mvp/
├── src/
│   ├── app/                      # Next.js 14 App Router
│   │   ├── (auth)/              # Rutas de autenticación
│   │   │   ├── login/
│   │   │   └── callback/
│   │   ├── dashboard/           # Dashboard protegido
│   │   ├── generate/            # Generación de documentos
│   │   │   └── [templateId]/
│   │   ├── api/                 # API Routes
│   │   │   ├── documents/
│   │   │   ├── payments/
│   │   │   └── webhooks/
│   │   ├── layout.tsx
│   │   └── page.tsx
│   │
│   ├── components/              # Componentes React
│   │   ├── ui/                  # shadcn/ui components
│   │   ├── DocumentWizard.tsx
│   │   ├── QuestionRenderer.tsx
│   │   └── TemplateCard.tsx
│   │
│   ├── lib/                     # Utilidades y configuración
│   │   ├── supabase.ts          # Cliente Supabase
│   │   ├── openai.ts            # Cliente OpenAI
│   │   ├── stripe.ts            # Cliente Stripe
│   │   ├── prompts/             # Prompts de IA
│   │   └── utils.ts
│   │
│   └── types/                   # TypeScript types
│       ├── template.ts
│       ├── document.ts
│       └── database.ts
│
├── public/                      # Assets estáticos
├── docs/                        # Documentación
│   ├── adr/                     # Architecture Decision Records
│   ├── api/                     # API documentation
│   └── architecture.md
│
├── scripts/                     # Scripts de utilidad
│   └── seed-templates.ts        # Seed de templates
│
├── tests/                       # Tests (próximamente)
│   ├── unit/
│   └── e2e/
│
├── .env.local.example           # Variables de entorno ejemplo
├── .gitignore
├── next.config.js
├── package.json
├── tsconfig.json
└── README.md
```

### Estructura Modular Recomendada
```
src/
├── modules/                    # Módulos de dominio
│   ├── auth/
│   │   ├── service.ts         # Lógica de negocio
│   │   ├── repository.ts      # Acceso a datos
│   │   └── types.ts           # Tipos/interfaces
│   │
│   ├── documents/
│   │   ├── service.ts
│   │   ├── repository.ts
│   │   └── types.ts
│   │
│   ├── ai-generation/
│   │   ├── service.ts         # ← Fácil de extraer
│   │   ├── openai.client.ts
│   │   └── types.ts
│   │
│   └── payments/
│       ├── service.ts
│       ├── stripe.client.ts
│       └── types.ts
│
└── app/                        # Next.js routes (thin controllers)
    └── api/
        ├── documents/
        │   └── route.ts        # ← Solo orquestación
        └── payments/
            └── route.ts

// Cada módulo es autocontenido
// Interfaces claras entre módulos
// Fácil de testear en aislamiento
// Fácil de extraer a servicio separado
```

---

## 🚀 Getting Started

### Prerequisitos

- Node.js 18.x o superior
- npm o yarn
- Cuenta en Supabase
- Cuenta en OpenAI
- Cuenta en Stripe (opcional para desarrollo)

### Instalación

1. **Clonar el repositorio**
```bash
git clone https://github.com/andersonaguiarc/legaia.git
cd legaia
```

2. **Instalar dependencias**
```bash
npm install
```

3. **Configurar variables de entorno**
```bash
cp .env.local.example .env.local
```

Editar `.env.local` con tus credenciales:
```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJxxx...
SUPABASE_SERVICE_ROLE_KEY=eyJxxx...

# OpenAI
OPENAI_API_KEY=sk-xxx...

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_xxx...
STRIPE_SECRET_KEY=sk_test_xxx...
STRIPE_WEBHOOK_SECRET=whsec_xxx...

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

4. **Ejecutar migraciones de base de datos**
```bash
# Conectar a Supabase SQL Editor y ejecutar schema.sql
# O usar CLI de Supabase (próximamente)
```

5. **Seed de templates (opcional)**
```bash
npm run seed
```

6. **Iniciar servidor de desarrollo**
```bash
npm run dev
```

Abrir [http://localhost:3000](http://localhost:3000) en tu navegador.

---

## 🗄️ Base de Datos

### Schema Principal

```sql
-- Templates de documentos
CREATE TABLE templates (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name TEXT NOT NULL,
  category TEXT NOT NULL,
  description TEXT,
  questions JSONB NOT NULL,
  prompt_template TEXT NOT NULL,
  base_price DECIMAL(10,2),
  created_at TIMESTAMP DEFAULT NOW()
);

-- Documentos generados
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id),
  template_id UUID REFERENCES templates(id),
  answers JSONB NOT NULL,
  content_html TEXT,
  file_url_pdf TEXT,
  status TEXT DEFAULT 'draft',
  payment_status TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- Suscripciones
CREATE TABLE subscriptions (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  user_id UUID REFERENCES auth.users(id),
  stripe_subscription_id TEXT,
  plan TEXT NOT NULL,
  status TEXT DEFAULT 'active',
  current_period_end TIMESTAMP
);
```

🔒 **Row Level Security (RLS)** habilitado en todas las tablas.

---

## 🧪 Testing

```bash
# Unit tests (próximamente)
npm run test

# E2E tests (próximamente)
npm run test:e2e

# Coverage (próximamente)
npm run test:coverage
```

---

## 📈 Roadmap

### ✅ Fase 0: Setup (Semana 1-2)
- [x] Configuración inicial del proyecto
- [x] Setup de Supabase
- [x] Autenticación básica
- [ ] Base de datos schema completo
- [ ] Wizard de documentos básico

### 🚧 Fase 1: MVP (Semana 3-6)
- [ ] Integración OpenAI
- [ ] Generación de documentos con IA
- [ ] Conversión a PDF
- [ ] Sistema de pagos (Stripe)
- [ ] Suscripciones
- [ ] 10 templates iniciales

### 📋 Fase 2: Beta (Semana 7-8)
- [ ] Landing page
- [ ] Testing E2E
- [ ] Deployment a producción
- [ ] 5 beta testers
- [ ] Monitoring y analytics

### 🎯 Fase 3: Launch (Mes 3)
- [ ] 20+ templates
- [ ] Sistema de revisión por abogados
- [ ] Dashboard mejorado
- [ ] Notificaciones por email
- [ ] SEO optimization

### 🚀 Futuro
- [ ] Chatbot legal
- [ ] Análisis de contratos
- [ ] Compliance checker
- [ ] Mobile app
- [ ] Expansión LATAM

---

## 👥 Equipo

### Co-Founders

**Anderson Aguiar** - CTO & Co-Founder  
🔧 Systems Engineer | 18 años de experiencia  
💻 Responsable de toda la arquitectura técnica y desarrollo  
🌐 [LinkedIn](https://www.linkedin.com/in/andersonaguiarc/) | [GitHub](https://github.com/andersonaguiarc)

**Katherine Aguiar** - Chief Legal Officer & Co-Founder  
⚖️ Abogada especialista en Derecho Laboral  
📋 Responsable de templates laborales y compliance  
🌐 [LinkedIn](#)

**María Juliana Grajales** - Chief Legal Officer & Co-Founder  
⚖️ Abogada especialista en Derecho Comercial  
📋 Responsable de templates comerciales y contratos  
🌐 [LinkedIn](#)

---

## 📊 Métricas del Proyecto

- **Stack moderno**: Next.js 14 + TypeScript + Supabase
- **IA de punta**: GPT-4 Turbo para generación
- **Arquitectura serverless**: Escalable desde día 1
- **Costo MVP**: $0-50/mes (primeros 3 meses)
- **Time to market**: 8 semanas
- **Objetivo Year 1**: $185K revenue, 280 usuarios

---

## 🔐 Seguridad

- ✅ Row Level Security (RLS) en Supabase
- ✅ Autenticación segura con Supabase Auth
- ✅ HTTPS en todas las comunicaciones
- ✅ Variables de entorno nunca expuestas al cliente
- ✅ Rate limiting en API routes
- ✅ Validación de inputs con Zod
- ✅ PCI compliance vía Stripe

---

## 📝 Licencia

Este proyecto es **propietario** y confidencial. Todos los derechos reservados.

**© 2026 LegaIA. Medellín, Colombia.**

---

## 🤝 Contribución

Este es un proyecto privado. Si eres parte del equipo, consulta `CONTRIBUTING.md` para lineamientos de desarrollo.

---

## 📞 Contacto

- **Website**: [legaia.digital](#) (próximamente)
- **Email**: contacto@legaia.digital
- **Location**: Medellín, Antioquia, Colombia

---

## 🙏 Agradecimientos

- [Next.js](https://nextjs.org/) por el framework increíble
- [Supabase](https://supabase.com/) por el backend instantáneo
- [OpenAI](https://openai.com/) por GPT-4
- [Vercel](https://vercel.com/) por el hosting gratuito
- [Platzi](https://platzi.com/) por la capacitación continua

---

<div align="center">

**Construido con ❤️ en Medellín, Colombia**

`Iniciado: 27 de Enero, 2026` | `Lanzamiento estimado: Marzo 2026`

</div>
