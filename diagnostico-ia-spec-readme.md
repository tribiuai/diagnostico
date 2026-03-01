# 🧠 AI Business Diagnostic App — Especificación Técnica Completa

## Nombre del Proyecto: **DiagnósticoIA** (versión FREE)

---

## 1. RESUMEN EJECUTIVO

Aplicación web que permite realizar un diagnóstico empresarial integral previo a la integración de soluciones de IA, evaluando hasta 10 áreas funcionales de la empresa a través de encuestas personalizadas generadas por IA, con reportes de madurez digital y recomendaciones estratégicas.

---

## 2. STACK TECNOLÓGICO

| Componente | Tecnología | Detalle |
|---|---|---|
| Frontend | React (Vite) + TailwindCSS | SPA servida por Express en producción |
| Backend | Node.js + Express | API REST, proxy a xAI, lógica de negocio |
| Base de datos | Supabase Free → Plan Pro | PostgreSQL + Auth (Google OAuth) + Storage (PDFs) |
| IA / LLM | Grok 4.1 Fast (xAI API) | Procesamiento de pre-diagnósticos y diagnóstico final |
| Automatización | n8n (ya en producción) | Notificaciones email, cierre automático de diagnósticos |
| Hosting | VPS Hostinger + Easypanel | Docker multi-stage, App from GitHub |
| Control de versiones | GitHub | Monorepo (backend/ + frontend/) |
| Generación PDF | React-PDF o jsPDF | Reportes de diagnóstico final |
| Dominio | diagnostico-ia.tribiu.com | DNS apuntando al VPS |

---

## 3. ROLES Y PERMISOS

### 3.1 Superadmin (Tú / Publitag)
- Dashboard global con todas las empresas registradas
- CRUD de empresas y diagnósticos
- Métricas generales de uso de la plataforma
- Gestión de configuración global (prompts, áreas funcionales, etc.)
- Acceso a todos los reportes generados

### 3.2 CEO / Gerente (Cliente)
- Registro de la empresa (formulario de pre-diagnóstico)
- Recibe código único y link personalizado
- Dashboard de avance: empleados que han respondido, porcentaje de completitud
- Vista previa parcial del diagnóstico (mientras se recopilan respuestas)
- Descarga del PDF final
- NO puede ver respuestas individuales (anonimato por área/departamento)

### 3.3 Empleado
- Ingresa con Google OAuth + código único de empresa
- Completa perfil básico (solo primer nombre, sin apellidos)
- Responde preguntas de pre-diagnóstico (perfil del empleado)
- Responde diagnóstico personalizado (generado por IA)
- Solo puede responder UNA vez
- No tiene acceso a resultados ni dashboard

---

## 4. ÁREAS FUNCIONALES EVALUABLES (Checklist)

El CEO selecciona las áreas que aplican a su empresa:

| # | Área Funcional | Descripción |
|---|---|---|
| 1 | Ventas | Gestión comercial, pipeline, cierre |
| 2 | Marketing | Estrategia digital, contenido, campañas |
| 3 | Contabilidad y Finanzas | Fiscal, impuestos, asientos contables, inversiones, préstamos LP |
| 4 | Alta Dirección / Liderazgo Estratégico | Gobierno corporativo, toma de decisiones, visión |
| 5 | Logística y Operaciones | Cadena de suministro, entrada y salida |
| 6 | Compras | Adquisiciones, proveedores, negociación |
| 7 | Inventarios | Control de stock, rotación, valorización |
| 8 | Producción | Manufactura, calidad, planificación |
| 9 | IT | Infraestructura, sistemas, soporte técnico |
| 10 | RRHH | Talento humano, capacitación, clima laboral |

---

## 5. FLUJO COMPLETO DE LA APLICACIÓN

### FASE 1: Registro de Empresa (CEO)

```
[CEO accede a la app] → [Formulario de pre-diagnóstico] → [IA procesa Input #1] 
→ [Se crea perfil de empresa en Supabase] → [Se genera código único con expiración] 
→ [n8n envía email al CEO con código + link personalizado]
```

**Formulario de Pre-Diagnóstico (Input #1 para IA):**

| # | Pregunta | Tipo |
|---|---|---|
| 1 | Nombre de la empresa | Texto abierto |
| 2 | ¿A qué se dedica la empresa? | Texto abierto |
| 3 | Número de empleados | Numérico |
| 4 | Áreas funcionales activas | Checklist (10 áreas) |
| 5 | ¿Tiene sistemas de información? | Checklist: CRM, ERP, Excel, Otros (campo abierto) |
| 6 | ¿Tiene la empresa experiencia en uso de IA? | Texto abierto |

**Procesamiento IA del Input #1:**
- La IA analiza el perfil y genera un resumen estructurado
- Identifica áreas de oportunidad iniciales
- Determina nivel de madurez tecnológica preliminar
- Este análisis se guarda en la BD como `company_ai_analysis`

### FASE 2: Distribución (CEO → Empleados)

```
[CEO recibe email con código + link] → [CEO distribuye internamente] 
→ [Empleados acceden al link]
```

- El código tiene expiración configurable (2-3 días por defecto)
- El link dirige a una landing con login de Google + campo para código
- El CEO puede ver en su dashboard cuántos empleados han ingresado

### FASE 3: Onboarding del Empleado

```
[Empleado ingresa con Google + código] → [Formulario de perfil] 
→ [IA procesa Input #2 "on the fly"] → [Se genera diagnóstico personalizado]
```

**Formulario de Perfil del Empleado (Input #2 para IA):**

| # | Pregunta | Tipo |
|---|---|---|
| 1 | Primer nombre | Texto |
| 2 | Años de experiencia | Numérico |
| 3 | Área de trabajo / posición actual | Selector (de las áreas marcadas por CEO) + texto |
| 4 | Principales atribuciones del puesto (para la empresa, el cliente y copartners internos) | Texto abierto |
| 5 | Principales aportes a la empresa desde su posición | Texto abierto |
| 6 | Herramientas y programas informáticos que utiliza | Checklist + texto abierto |
| 7 | Satisfacción general como empleado | Escala 1-5 + texto abierto opcional |

**Procesamiento IA del Input #2 (on the fly):**
- La IA analiza el perfil del empleado en contexto con el Input #1
- Genera un pre-diagnóstico individual
- Se almacena como `employee_ai_profile`

### FASE 4: Diagnóstico del Empleado

```
[IA genera 20 preguntas personalizadas] → [Empleado responde] 
→ [Respuestas se guardan en BD] → [Estado: "completado"]
```

**Estructura de las 20 preguntas generadas por IA:**

| Tipo | Cantidad | Naturaleza | Base |
|---|---|---|---|
| Escala Likert (1-5) | 10 | Base común para todos los empleados de la empresa | Perfil de empresa (Input #1) + área funcional |
| Cerradas (Sí/No/Opción múltiple) | 5 | Adecuadas al área funcional del empleado | Área funcional específica |
| Abiertas (texto libre) | 5 | Personalizadas al perfil del empleado | Input #1 + Input #2 combinados |

**Reglas para generación de preguntas:**
- Las 10 preguntas Likert deben cubrir las áreas funcionales seleccionadas por el CEO
- Las 5 cerradas deben enfocarse en el área específica del empleado
- Las 5 abiertas deben explorar la experiencia del empleado con tecnología/IA en su contexto
- Cada pregunta debe alinearse con el objetivo de diagnosticar madurez y oportunidades de IA

### FASE 5: Cierre y Generación de Diagnóstico

```
[Expira el código (2-3 días)] → [Se cierra recopilación] 
→ [IA analiza TODA la información] → [Genera diagnóstico final] 
→ [PDF se genera y almacena] → [n8n notifica al CEO] 
→ [CEO descarga PDF desde dashboard]
```

**Estructura del PDF de Diagnóstico Final:**

1. **Portada**: Nombre empresa, fecha, logo DiagnósticoIA
2. **Resumen Ejecutivo**: Hallazgos clave en 1 página
3. **Índice de Madurez Digital Global**: Score general (escala 1-5) con visualización tipo radar/spider chart
4. **Diagnóstico por Área Funcional** (solo áreas seleccionadas):
   - Índice de madurez del área (1-5)
   - Hallazgos clave (anónimos, agrupados por departamento)
   - Fortalezas identificadas
   - Brechas y oportunidades
   - Recomendaciones de soluciones IA específicas para el área
5. **Análisis de Infraestructura Tecnológica**: Estado actual de sistemas y herramientas
6. **Análisis de Capital Humano**: Nivel de adopción tecnológica, disposición al cambio
7. **Recomendaciones Estratégicas de IA por Área**: Soluciones específicas, herramientas sugeridas
8. **Roadmap Sugerido de Implementación**: Fases, prioridades, quick wins
9. **Anexos**: Metodología, glosario

---

## 6. MODELO DE BASE DE DATOS (Supabase)

### Tablas Principales

```sql
-- ============================================
-- EMPRESAS
-- ============================================
CREATE TABLE companies (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  description TEXT,                    -- A qué se dedica
  employee_count INTEGER,
  functional_areas TEXT[],             -- Array de áreas seleccionadas
  info_systems JSONB,                  -- {crm: bool, erp: bool, excel: bool, others: "..."}
  ai_experience TEXT,                  -- Experiencia con IA
  ai_pre_diagnosis JSONB,             -- Análisis IA del Input #1
  unique_code TEXT UNIQUE NOT NULL,    -- Código único generado
  code_expires_at TIMESTAMPTZ,        -- Expiración del código
  diagnosis_status TEXT DEFAULT 'open', -- open | closed | completed
  final_diagnosis JSONB,              -- Diagnóstico final generado por IA
  pdf_url TEXT,                        -- URL del PDF en Supabase Storage
  ceo_email TEXT NOT NULL,
  ceo_name TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  closed_at TIMESTAMPTZ
);

-- ============================================
-- USUARIOS (todos los roles)
-- ============================================
CREATE TABLE users (
  id UUID REFERENCES auth.users(id) PRIMARY KEY,
  email TEXT NOT NULL,
  first_name TEXT,
  role TEXT NOT NULL DEFAULT 'employee', -- superadmin | ceo | employee
  company_id UUID REFERENCES companies(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- PERFILES DE EMPLEADOS
-- ============================================
CREATE TABLE employee_profiles (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  company_id UUID REFERENCES companies(id),
  first_name TEXT,                     -- Solo primer nombre (anonimato)
  years_experience INTEGER,
  work_area TEXT,                      -- Área de trabajo / posición
  main_duties TEXT,                    -- Atribuciones del puesto
  main_contributions TEXT,             -- Aportes a la empresa
  tools_used JSONB,                    -- Herramientas y programas
  satisfaction_score INTEGER,          -- 1-5
  satisfaction_comment TEXT,
  ai_profile_analysis JSONB,          -- Análisis IA del Input #2
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- PREGUNTAS DEL DIAGNÓSTICO (generadas por IA)
-- ============================================
CREATE TABLE diagnosis_questions (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  company_id UUID REFERENCES companies(id),
  employee_id UUID REFERENCES employee_profiles(id),
  question_text TEXT NOT NULL,
  question_type TEXT NOT NULL,         -- likert | closed | open
  question_category TEXT,              -- Área funcional asociada
  options JSONB,                       -- Opciones para preguntas cerradas
  question_order INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- RESPUESTAS DEL DIAGNÓSTICO
-- ============================================
CREATE TABLE diagnosis_responses (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  question_id UUID REFERENCES diagnosis_questions(id),
  employee_id UUID REFERENCES employee_profiles(id),
  company_id UUID REFERENCES companies(id),
  response_value TEXT,                 -- Valor de la respuesta (1-5, opción, o texto)
  response_type TEXT,                  -- likert | closed | open
  work_area TEXT,                      -- Para agrupar por área en el reporte
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE(question_id, employee_id)    -- Un empleado, una respuesta por pregunta
);

-- ============================================
-- LOG DE PROCESAMIENTO IA
-- ============================================
CREATE TABLE ai_processing_log (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  company_id UUID REFERENCES companies(id),
  employee_id UUID REFERENCES employee_profiles(id),
  process_type TEXT,                   -- pre_diagnosis_company | pre_diagnosis_employee | question_generation | final_diagnosis
  input_data JSONB,
  output_data JSONB,
  model_used TEXT DEFAULT 'grok-4.1-fast',
  tokens_used INTEGER,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

### Row Level Security (RLS)

```
- Superadmin: acceso total a todas las tablas
- CEO: solo ve datos de su company_id
- Empleado: solo ve/edita sus propios registros
- Respuestas: empleado puede crear pero no editar después de enviar
```

---

## 7. DISEÑO DEL PROMPT MAESTRO PARA LA IA (Grok 4.1 Fast)

### 7.1 PROMPT: Análisis de Pre-Diagnóstico de Empresa (Input #1)

```
SISTEMA: Eres un consultor experto en transformación digital e integración de
Inteligencia Artificial en empresas. Tu tarea es analizar la información de una
empresa para generar un pre-diagnóstico que servirá como base para un diagnóstico
completo de madurez digital.

CONTEXTO: Has recibido la información del formulario de registro de una empresa
que busca integrar soluciones de IA en sus operaciones.

DATOS DE LA EMPRESA:
- Nombre: {{company_name}}
- Giro/Actividad: {{company_description}}
- Número de empleados: {{employee_count}}
- Áreas funcionales activas: {{functional_areas}}
- Sistemas de información actuales: {{info_systems}}
- Experiencia con IA: {{ai_experience}}

INSTRUCCIONES:
Genera un análisis estructurado en formato JSON con los siguientes campos:

{
  "resumen_empresa": "Descripción breve del tipo de empresa y su contexto",
  "nivel_madurez_tecnologica_preliminar": {
    "score": 1-5,
    "justificacion": "..."
  },
  "areas_oportunidad_ia": [
    {
      "area": "nombre del área funcional",
      "oportunidad": "descripción de la oportunidad",
      "prioridad": "alta|media|baja",
      "complejidad_implementacion": "alta|media|baja"
    }
  ],
  "brechas_tecnologicas": ["lista de brechas identificadas"],
  "fortalezas_detectadas": ["lista de fortalezas"],
  "riesgos_potenciales": ["riesgos a considerar"],
  "recomendaciones_iniciales": ["recomendaciones preliminares"],
  "perfil_industria": "contexto de la industria respecto a la IA",
  "preguntas_clave_a_explorar": [
    "preguntas que deberían explorarse con los empleados"
  ]
}

IMPORTANTE:
- Sé objetivo y específico al contexto de la empresa
- Las recomendaciones deben ser accionables
- Considera el tamaño de la empresa y sus recursos
- Identifica quick wins vs proyectos de largo plazo
- Responde SOLO con el JSON, sin texto adicional
```

### 7.2 PROMPT: Análisis de Perfil del Empleado (Input #2)

```
SISTEMA: Eres un consultor de transformación digital analizando perfiles de
empleados para un diagnóstico de madurez de IA empresarial.

CONTEXTO EMPRESA:
{{company_ai_analysis}} (resultado del Input #1)

PERFIL DEL EMPLEADO:
- Nombre: {{first_name}}
- Años de experiencia: {{years_experience}}
- Área/Posición: {{work_area}}
- Atribuciones del puesto: {{main_duties}}
- Aportes a la empresa: {{main_contributions}}
- Herramientas que usa: {{tools_used}}
- Satisfacción (1-5): {{satisfaction_score}}
- Comentario de satisfacción: {{satisfaction_comment}}

INSTRUCCIONES:
Genera un análisis del perfil del empleado en formato JSON:

{
  "resumen_perfil": "Descripción del perfil profesional y su rol",
  "nivel_digitalizacion_individual": {
    "score": 1-5,
    "justificacion": "..."
  },
  "areas_impacto_ia": [
    "áreas donde la IA podría impactar su trabajo"
  ],
  "herramientas_ia_sugeridas": [
    "herramientas específicas que podrían beneficiarle"
  ],
  "brecha_competencias_digitales": "análisis de la brecha",
  "potencial_adopcion": "alto|medio|bajo",
  "aspectos_a_explorar": [
    "temas específicos para profundizar en el diagnóstico"
  ]
}

IMPORTANTE:
- Contextualiza con el perfil de la empresa
- Sé específico al área funcional del empleado
- Responde SOLO con el JSON, sin texto adicional
```

### 7.3 PROMPT: Generación de Preguntas del Diagnóstico

```
SISTEMA: Eres un consultor experto diseñando diagnósticos de madurez digital.
Genera preguntas personalizadas para evaluar la preparación de una empresa
para integrar IA.

CONTEXTO EMPRESA (Input #1):
{{company_ai_analysis}}

PERFIL EMPLEADO (Input #2):
{{employee_ai_profile}}

ÁREAS FUNCIONALES A EVALUAR:
{{functional_areas}}

ÁREA DEL EMPLEADO:
{{employee_work_area}}

INSTRUCCIONES:
Genera exactamente 20 preguntas distribuidas así:

1. **10 PREGUNTAS LIKERT (escala 1-5)** - Base común:
   - Deben cubrir las áreas funcionales seleccionadas por la empresa
   - Escala: 1=Totalmente en desacuerdo, 2=En desacuerdo, 3=Neutral,
     4=De acuerdo, 5=Totalmente de acuerdo
   - Enfocadas en procesos actuales, uso de tecnología, y disposición al cambio

2. **5 PREGUNTAS CERRADAS** - Específicas al área funcional:
   - Formato: opción múltiple (3-4 opciones) o Sí/No
   - Deben enfocarse en el área funcional específica del empleado
   - Evaluar prácticas actuales y necesidades operativas del área

3. **5 PREGUNTAS ABIERTAS** - Personalizadas:
   - Basadas en el perfil de la empresa Y el perfil del empleado
   - Explorar experiencias, necesidades y visión del empleado
   - Deben generar insights accionables para el diagnóstico

Formato JSON de respuesta:
{
  "questions": [
    {
      "id": 1,
      "type": "likert",
      "category": "área funcional relacionada",
      "text": "texto de la pregunta",
      "scale": {"min": 1, "max": 5, "labels": {"1": "...", "5": "..."}}
    },
    {
      "id": 11,
      "type": "closed",
      "category": "área funcional",
      "text": "texto de la pregunta",
      "options": ["Opción A", "Opción B", "Opción C"]
    },
    {
      "id": 16,
      "type": "open",
      "category": "área funcional o transversal",
      "text": "texto de la pregunta",
      "max_length": 500
    }
  ]
}

REGLAS:
- Las preguntas deben ser claras y sin ambigüedades
- Usar lenguaje profesional pero accesible
- Evitar jerga técnica excesiva
- Cada pregunta debe aportar valor diagnóstico
- Las preguntas abiertas deben invitar a respuestas reflexivas
- Todas las preguntas en ESPAÑOL
- Responde SOLO con el JSON, sin texto adicional
```

### 7.4 PROMPT: Diagnóstico Final

```
SISTEMA: Eres un consultor senior de transformación digital. Genera un diagnóstico
empresarial completo basado en toda la información recopilada. El reporte es para
el CEO/Gerente y debe ser estratégico, accionable y profesional.

DATOS DE LA EMPRESA:
{{company_data}}

PRE-DIAGNÓSTICO EMPRESA (Input #1):
{{company_ai_analysis}}

PERFILES DE EMPLEADOS ANALIZADOS:
{{employee_profiles_summary}} (agrupados por área, anónimos)

RESPUESTAS DEL DIAGNÓSTICO:
{{all_responses_aggregated}} (agrupadas por área funcional, anónimas)

ÁREAS FUNCIONALES EVALUADAS:
{{functional_areas}}

INSTRUCCIONES:
Genera un diagnóstico completo en formato JSON con la siguiente estructura:

{
  "resumen_ejecutivo": {
    "titulo": "Diagnóstico de Madurez Digital y Preparación para IA",
    "empresa": "{{company_name}}",
    "fecha": "{{date}}",
    "hallazgos_clave": ["lista de 3-5 hallazgos principales"],
    "conclusion_general": "párrafo resumen"
  },
  "indice_madurez_global": {
    "score": 1-5 (con decimal),
    "nivel": "Inicial|Básico|Intermedio|Avanzado|Líder",
    "descripcion": "descripción del nivel actual"
  },
  "diagnostico_por_area": [
    {
      "area": "nombre del área",
      "indice_madurez": {
        "score": 1-5,
        "nivel": "..."
      },
      "participantes": número,
      "hallazgos": ["hallazgos del área"],
      "fortalezas": ["fortalezas identificadas"],
      "brechas": ["brechas y debilidades"],
      "sentimiento_general": "positivo|neutro|negativo",
      "soluciones_ia_recomendadas": [
        {
          "solucion": "nombre/tipo de solución",
          "descripcion": "qué hace y cómo ayuda",
          "impacto_esperado": "alto|medio|bajo",
          "complejidad": "alta|media|baja",
          "herramientas_sugeridas": ["herramientas específicas"],
          "estimacion_implementacion": "timeframe"
        }
      ]
    }
  ],
  "analisis_infraestructura": {
    "estado_actual": "descripción",
    "sistemas_detectados": ["lista"],
    "brechas_tecnologicas": ["lista"],
    "recomendaciones": ["lista"]
  },
  "analisis_capital_humano": {
    "nivel_adopcion_tecnologica": "score 1-5",
    "disposicion_al_cambio": "alta|media|baja",
    "brechas_competencias": ["lista"],
    "plan_upskilling": ["recomendaciones"]
  },
  "roadmap_implementacion": {
    "fase_1_quick_wins": {
      "duracion": "1-3 meses",
      "acciones": ["lista de acciones rápidas"]
    },
    "fase_2_fundamentos": {
      "duracion": "3-6 meses",
      "acciones": ["lista de acciones"]
    },
    "fase_3_escalamiento": {
      "duracion": "6-12 meses",
      "acciones": ["lista de acciones"]
    },
    "fase_4_optimizacion": {
      "duracion": "12+ meses",
      "acciones": ["lista de acciones"]
    }
  },
  "proximos_pasos": ["lista de acciones inmediatas recomendadas"]
}

REGLAS:
- Todo en ESPAÑOL
- Ser específico y accionable (no genérico)
- Las respuestas de empleados son ANÓNIMAS, agrupar por área/departamento
- Las soluciones de IA deben ser realistas para el tamaño de la empresa
- El roadmap debe considerar los recursos disponibles
- Incluir tanto soluciones de IA gratuitas como de pago
- Responde SOLO con el JSON, sin texto adicional
```

---

## 8. ARQUITECTURA DE COMPONENTES (Monorepo)

### Estructura de Carpetas del Proyecto

```
diagnostico-ia/                        -- Raíz del monorepo (GitHub repo)
├── backend/
│   ├── config/
│   │   ├── supabase.js                -- Cliente Supabase (server-side con service key)
│   │   └── xai.js                     -- Configuración xAI API
│   ├── routes/
│   │   ├── auth.js                    -- Google OAuth callbacks, validación código
│   │   ├── companies.js               -- CRUD empresas, generación código
│   │   ├── employees.js               -- Perfiles y respuestas empleados
│   │   ├── diagnosis.js               -- Generación preguntas, cierre, diagnóstico final
│   │   ├── admin.js                   -- Endpoints superadmin
│   │   └── webhooks.js                -- Endpoints para n8n
│   ├── services/
│   │   ├── aiService.js               -- Llamadas a Grok/xAI API (prompts)
│   │   ├── codeGenerator.js           -- Generación de códigos únicos
│   │   ├── pdfGenerator.js            -- Generación de PDFs con diagnóstico
│   │   └── diagnosisEngine.js         -- Lógica de procesamiento del diagnóstico
│   ├── middleware/
│   │   ├── auth.js                    -- Validación JWT + roles
│   │   ├── roleGuard.js               -- Protección por rol (superadmin, ceo, employee)
│   │   └── rateLimiter.js             -- Throttling de requests
│   ├── server.js                      -- Express server + sirve frontend en producción
│   ├── package.json
│   └── .env                           -- Variables de entorno (NO va al repo)
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── auth/
│   │   │   │   ├── GoogleLoginButton.jsx
│   │   │   │   ├── CodeInput.jsx
│   │   │   │   └── ProtectedRoute.jsx
│   │   │   ├── superadmin/
│   │   │   │   ├── Dashboard.jsx
│   │   │   │   ├── CompanyList.jsx
│   │   │   │   ├── CompanyDetail.jsx
│   │   │   │   └── GlobalMetrics.jsx
│   │   │   ├── ceo/
│   │   │   │   ├── CompanyRegistration.jsx
│   │   │   │   ├── CEODashboard.jsx
│   │   │   │   ├── DiagnosisPreview.jsx
│   │   │   │   └── ReportDownload.jsx
│   │   │   ├── employee/
│   │   │   │   ├── EmployeeOnboarding.jsx
│   │   │   │   ├── DiagnosisQuestions.jsx
│   │   │   │   ├── LikertScale.jsx
│   │   │   │   ├── ClosedQuestion.jsx
│   │   │   │   ├── OpenQuestion.jsx
│   │   │   │   └── CompletionScreen.jsx
│   │   │   ├── shared/
│   │   │   │   ├── LoadingSpinner.jsx
│   │   │   │   ├── ProgressBar.jsx
│   │   │   │   ├── Header.jsx
│   │   │   │   ├── Footer.jsx
│   │   │   │   └── TermsPrivacy.jsx
│   │   │   └── pdf/
│   │   │       ├── DiagnosisReport.jsx
│   │   │       ├── RadarChart.jsx
│   │   │       └── AreaCard.jsx
│   │   ├── hooks/
│   │   │   ├── useAuth.js
│   │   │   ├── useCompany.js
│   │   │   ├── useDiagnosis.js
│   │   │   └── useAI.js
│   │   ├── services/
│   │   │   └── api.js                 -- Axios/fetch wrapper al backend
│   │   ├── contexts/
│   │   │   ├── AuthContext.jsx
│   │   │   └── DiagnosisContext.jsx
│   │   ├── pages/
│   │   │   ├── Landing.jsx
│   │   │   ├── Login.jsx
│   │   │   ├── SuperadminPanel.jsx
│   │   │   ├── CEOPanel.jsx
│   │   │   ├── EmployeeFlow.jsx
│   │   │   ├── Terms.jsx
│   │   │   └── NotFound.jsx
│   │   ├── utils/
│   │   │   ├── constants.js
│   │   │   └── helpers.js
│   │   ├── App.jsx
│   │   └── main.jsx
│   ├── index.html
│   ├── vite.config.js
│   ├── tailwind.config.js
│   └── package.json
├── Dockerfile                          -- Multi-stage: build React + run Express
├── .dockerignore
├── .gitignore
├── .env.example                        -- Template de variables (SÍ va al repo)
└── README.md
```

### Variables de Entorno (backend/.env)

```env
# ============================================
# SERVER
# ============================================
PORT=3001
NODE_ENV=production

# ============================================
# SUPABASE
# ============================================
SUPABASE_URL=https://xxxxx.supabase.co
SUPABASE_ANON_KEY=eyJxxxxx
SUPABASE_SERVICE_KEY=eyJxxxxx              # Server-side only, nunca en frontend

# ============================================
# xAI (Grok)
# ============================================
XAI_API_KEY=xai-xxxxx
XAI_MODEL=grok-4.1-fast
XAI_BASE_URL=https://api.x.ai/v1

# ============================================
# GOOGLE OAUTH (vía Supabase Auth)
# ============================================
GOOGLE_CLIENT_ID=xxxxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=xxxxx

# ============================================
# n8n WEBHOOKS
# ============================================
N8N_WEBHOOK_SEND_EMAIL=https://marfund-ia-n8n.9867lv.easypanel.host/webhook/diag-send-email
N8N_WEBHOOK_CLOSE_DIAGNOSIS=https://marfund-ia-n8n.9867lv.easypanel.host/webhook/diag-close
N8N_WEBHOOK_PDF_READY=https://marfund-ia-n8n.9867lv.easypanel.host/webhook/diag-pdf-ready

# ============================================
# APP CONFIG
# ============================================
APP_URL=https://diagnostico-ia.tribiu.com
CODE_EXPIRATION_HOURS=72
MAX_CONCURRENT_COMPANIES=10

# ============================================
# SESSION / JWT
# ============================================
SESSION_SECRET=cambiar-en-produccion-xxxxx
```

### Variables de Entorno (frontend - vite.config.js proxy)

```env
# El frontend NO tiene API keys directas
# Todas las llamadas van al backend vía /api/*
# Vite en desarrollo hace proxy a localhost:3001
VITE_APP_URL=https://diagnostico-ia.tribiu.com
```

---

## 9. FLUJOS DE n8n

### 9.1 Flujo: Enviar Email con Código al CEO
```
Trigger: Webhook (POST /send-code-email)
→ Recibe: {ceo_email, ceo_name, company_name, unique_code, link}
→ Nodo Email (SMTP/Gmail): Envía email con template HTML
→ Responde: {success: true}
```

### 9.2 Flujo: Cerrar Diagnóstico por Expiración
```
Trigger: Cron (cada hora)
→ Query Supabase: empresas con code_expires_at < NOW() AND status = 'open'
→ Para cada empresa:
  → Actualizar status = 'closed'
  → Webhook interno: disparar generación de diagnóstico final
  → Email al CEO: "Su diagnóstico está siendo procesado"
```

### 9.3 Flujo: Notificar PDF Listo
```
Trigger: Webhook (POST /diagnosis-complete)
→ Recibe: {ceo_email, company_name, pdf_url}
→ Email al CEO: "Su diagnóstico está listo para descarga"
→ Incluye link directo al dashboard
```

---

## 10. SEGURIDAD Y CONSIDERACIONES

### Autenticación
- Google OAuth 2.0 vía Supabase Auth
- Validación de código único en cada request del empleado
- JWT tokens con expiración
- RLS (Row Level Security) en todas las tablas

### Privacidad
- Empleados: solo primer nombre, sin apellidos
- Respuestas agrupadas por área/departamento en reportes
- CEO NO puede ver respuestas individuales
- Página de términos y condiciones obligatoria antes del diagnóstico

### Rate Limiting
- Máximo 10 empresas concurrentes
- Throttling en llamadas a xAI API
- Cola de procesamiento para diagnósticos finales

### Datos
- Backups automáticos de Supabase
- PDFs almacenados en Supabase Storage con URLs firmadas
- Logs de procesamiento de IA para auditoría

## 11. INFRAESTRUCTURA Y DEPLOYMENT

### 11.1 Arquitectura de Servicios en Easypanel

```
┌─────────────────────────────────────────────────────────────┐
│                    VPS HOSTINGER                            │
│                    (Easypanel)                              │
│                                                             │
│  ┌─────────────────────────┐  ┌──────────────────────────┐ │
│  │  diagnostico-ia (App)   │  │  n8n (ya existente)      │ │
│  │  ─────────────────────  │  │  ────────────────────    │ │
│  │  Docker multi-stage     │  │  Email notifications     │ │
│  │  ┌───────────────────┐  │  │  Cron jobs (cierre)      │ │
│  │  │ Express (port 3001)│  │  │  Webhooks               │ │
│  │  │ + React build      │  │  │                          │ │
│  │  │ (static files)     │  │  │  marfund-ia-n8n.         │ │
│  │  └───────────────────┘  │  │  9867lv.easypanel.host   │ │
│  │                         │  │                          │ │
│  │  diagnostico-ia.        │  └──────────────────────────┘ │
│  │  tribiu.com             │                               │
│  └────────────┬────────────┘                               │
│               │                                             │
└───────────────┼─────────────────────────────────────────────┘
                │
    ┌───────────┼──────────────────────┐
    │           │                      │
    ▼           ▼                      ▼
┌────────┐ ┌────────────┐  ┌──────────────────┐
│Supabase│ │ xAI API    │  │ Google OAuth      │
│(Cloud) │ │ (Grok 4.1) │  │ (Supabase Auth)   │
│        │ │            │  │                    │
│ - DB   │ │ Prompts:   │  │ Gmail +            │
│ - Auth │ │ Input #1   │  │ Google Workspace   │
│ - Store│ │ Input #2   │  └──────────────────┘
│   (PDF)│ │ Questions  │
│        │ │ Diagnosis  │
└────────┘ └────────────┘
```

### 11.2 Dockerfile (Multi-stage: React build + Express server)

```dockerfile
# ── STAGE 1: Build React Frontend ──────────────────────────
FROM node:20-alpine AS frontend-builder
WORKDIR /app/frontend
COPY frontend/package*.json ./
RUN npm ci
COPY frontend/ .
RUN npm run build

# ── STAGE 2: Backend Node.js + sirve el frontend ───────────
FROM node:20-alpine AS production
WORKDIR /app

# Instalar dependencias del backend
COPY backend/package*.json ./
RUN npm ci --only=production

# Copiar backend
COPY backend/ .

# Copiar el build del frontend al backend para servirlo estáticamente
COPY --from=frontend-builder /app/frontend/dist ./public

# Puerto expuesto
EXPOSE 3001

# Healthcheck para Easypanel
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget --spider http://localhost:3001/api/health || exit 1

# Arrancar la app
CMD ["node", "server.js"]
```

### 11.3 Express Server (backend/server.js) — Producción

```javascript
const express = require('express');
const path = require('path');
const app = express();

// Middleware
app.use(express.json());

// API Routes
app.use('/api/auth', require('./routes/auth'));
app.use('/api/companies', require('./routes/companies'));
app.use('/api/employees', require('./routes/employees'));
app.use('/api/diagnosis', require('./routes/diagnosis'));
app.use('/api/admin', require('./routes/admin'));
app.use('/api/webhooks', require('./routes/webhooks'));

// Health check para Easypanel
app.get('/api/health', (req, res) => res.json({ status: 'ok' }));

// Servir React en producción
if (process.env.NODE_ENV === 'production') {
  app.use(express.static(path.join(__dirname, 'public')));
  app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
  });
}

const PORT = process.env.PORT || 3001;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

### 11.4 Configuración en Easypanel

**Crear servicio:**
1. Easypanel → **"+ New Project"** → nombre: `diagnostico-ia`
2. **"+ Service"** → **"App"** → Source: **GitHub**
3. Conectar repo → Branch: `main`
4. Build: Dockerfile (detecta automáticamente)
5. Port: `3001`

**Variables de entorno en Easypanel:**
Copiar todas las variables del `backend/.env` en la sección "Environment Variables" del servicio.

**Dominio:**
1. En Easypanel → servicio → pestaña **"Domains"**
2. Agregar: `diagnostico-ia.tribiu.com`
3. Habilitar SSL (Let's Encrypt automático)

**DNS en Hostinger/Registrador:**
```
Tipo: A
Nombre: diagnostico-ia.tribiu.com
Valor: [IP del VPS]
TTL: 3600
```

### 11.5 Flujo GitHub → Easypanel (CI/CD)

```
Tu máquina local (VS Code)
  git checkout -b feature/nueva-tarea
  [codeas y pruebas localmente]
  git add .
  git commit -m "feat: descripción"
  git push origin feature/nueva-tarea
       │
       ▼
  GitHub → Pull Request → merge a main
       │
       ▼
  Easypanel detecta cambio en main → auto-deploy
       │
       ▼
  Docker build multi-stage (~2-3 minutos)
       │
       ▼
  https://diagnostico-ia.tribiu.com actualizado ✅
```

**Estrategia de branches:**
```
main          ← producción (lo que Easypanel despliega automáticamente)
  └── develop ← desarrollo e integración
        └── feature/nombre-tarea ← cada nueva función
```

### 11.6 Archivos de Configuración del Repo

**.gitignore:**
```
node_modules/
dist/
build/
.env
.env.local
.env.production
*.log
.DS_Store
```

**.env.example** (SÍ va al repo, sin valores reales):
```env
PORT=3001
NODE_ENV=development
SUPABASE_URL=
SUPABASE_ANON_KEY=
SUPABASE_SERVICE_KEY=
XAI_API_KEY=
XAI_MODEL=grok-4.1-fast
XAI_BASE_URL=https://api.x.ai/v1
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
N8N_WEBHOOK_SEND_EMAIL=
N8N_WEBHOOK_CLOSE_DIAGNOSIS=
N8N_WEBHOOK_PDF_READY=
APP_URL=https://diagnostico-ia.tribiu.com
CODE_EXPIRATION_HOURS=72
MAX_CONCURRENT_COMPANIES=10
SESSION_SECRET=
```

**.dockerignore:**
```
node_modules/
.git/
.env
*.md
.vscode/
```

### 11.7 Desarrollo Local

```bash
# Clonar el repo
git clone https://github.com/tu-usuario/diagnostico-ia.git
cd diagnostico-ia

# Setup backend
cd backend
cp ../.env.example .env        # Editar con tus valores locales
npm install
npm run dev                    # Express en localhost:3001

# Setup frontend (otra terminal)
cd frontend
npm install
npm run dev                    # Vite en localhost:5173 con proxy a :3001
```

**vite.config.js (proxy en desarrollo):**
```javascript
export default defineConfig({
  plugins: [react()],
  server: {
    port: 5173,
    proxy: {
      '/api': {
        target: 'http://localhost:3001',
        changeOrigin: true
      }
    }
  }
})
```

### 11.8 Supabase — Configuración

**Plan Free (inicio):**
- 500 MB base de datos
- 1 GB Storage (suficiente para PDFs iniciales)
- 50,000 Auth users/mes
- 2 GB bandwidth

**Migración a Plan Pro cuando:**
- Superes 500 MB en BD
- Necesites más de 1 GB en Storage
- Requieras backups automáticos diarios
- Necesites soporte prioritario

**Configuración Auth (Google OAuth):**
1. Supabase Dashboard → Authentication → Providers → Google
2. Habilitar Google
3. Agregar Client ID y Client Secret de Google Cloud Console
4. Redirect URL: `https://diagnostico-ia.tribiu.com/auth/callback`

**Storage Bucket para PDFs:**
```sql
-- Crear bucket en Supabase
INSERT INTO storage.buckets (id, name, public) 
VALUES ('diagnosis-pdfs', 'diagnosis-pdfs', false);

-- RLS: Solo el CEO de la empresa puede descargar su PDF
CREATE POLICY "CEO can download own company PDFs"
ON storage.objects FOR SELECT
USING (
  bucket_id = 'diagnosis-pdfs' 
  AND auth.uid() IN (
    SELECT u.id FROM users u 
    JOIN companies c ON u.company_id = c.id 
    WHERE c.id = (storage.foldername(name))[1]::uuid
    AND u.role = 'ceo'
  )
);
```

### 11.9 n8n — Integración con Instancia Existente

Tu n8n ya corre en: `marfund-ia-n8n.9867lv.easypanel.host`

**Webhooks a crear en n8n:**

| Webhook | Path | Método | Trigger |
|---|---|---|---|
| Enviar código al CEO | `/webhook/diag-send-email` | POST | Registro de empresa completado |
| Cerrar diagnóstico | `/webhook/diag-close` | CRON | Cada hora, revisa expiración |
| Notificar PDF listo | `/webhook/diag-pdf-ready` | POST | Diagnóstico final generado |

**Credenciales necesarias en n8n:**
- SMTP (Gmail o servicio email) para envío de correos
- Supabase (para queries de cierre automático)

---

## 12. SKILLS NECESARIAS PARA DESARROLLO

### 12.1 Skills por Área

| Skill | Tecnología | Prioridad |
|---|---|---|
| Frontend React | React + Vite + TailwindCSS | Alta |
| Autenticación | Supabase Auth + Google OAuth | Alta |
| Base de datos | Supabase PostgreSQL + RLS | Alta |
| Integración IA | xAI API (Grok 4.1 Fast) | Alta |
| Prompt Engineering | Diseño y refinamiento de prompts | Alta |
| Generación PDF | React-PDF o jsPDF | Alta |
| Automatización | n8n workflows | Media |
| UI/UX | Diseño responsivo, formularios dinámicos | Media |
| Charts/Gráficos | Recharts o Chart.js (radar charts) | Media |
| DevOps | Docker + Easypanel + GitHub | Media |
| Email Templates | HTML emails responsivos | Baja |
| Testing | Jest + React Testing Library | Baja (v1) |

### 12.2 Antigravity Skills (skills.sh) — Script de Instalación

Ejecutar en la raíz del monorepo `diagnostico-ia/` antes de iniciar el desarrollo con Claude/Antigravity.

```bash
# ============================================
# DIAGNOSTICO-IA — Instalación de Skills
# Ejecutar en la raíz del proyecto: diagnostico-ia/
# ============================================

# ── 🎨 Frontend — React + TailwindCSS ──────────────────────
npx skills add vercel-labs/agent-skills/vercel-react-best-practices
npx skills add vercel-labs/agent-skills/web-design-guidelines
npx skills add vercel-labs/agent-skills/vercel-composition-patterns
npx skills add anthropics/skills/frontend-design
npx skills add wshobson/agents/tailwind-design-system
npx skills add wshobson/agents/react-state-management

# ── ⚙️ Backend — Node.js + Express ─────────────────────────
npx skills add wshobson/agents/nodejs-backend-patterns
npx skills add wshobson/agents/api-design-principles
npx skills add wshobson/agents/error-handling-patterns
npx skills add wshobson/agents/auth-implementation-patterns

# ── 🗄️ Base de Datos — Supabase / PostgreSQL ───────────────
npx skills add supabase/agent-skills/supabase-postgres-best-practices
npx skills add wshobson/agents/postgresql-table-design
npx skills add wshobson/agents/sql-optimization-patterns

# ── 🔐 Autenticación — Google OAuth ────────────────────────
npx skills add better-auth/skills/better-auth-best-practices

# ── 🔀 Git + GitHub + Docker ───────────────────────────────
npx skills add wshobson/agents/git-advanced-workflows
npx skills add github/awesome-copilot/git-commit
npx skills add obra/superpowers/finishing-a-development-branch

# ── 📧 Email ───────────────────────────────────────────────
npx skills add resend/email-best-practices/email-best-practices

# ── 📋 Planificación y Calidad de Código ───────────────────
npx skills add obra/superpowers/writing-plans
npx skills add obra/superpowers/executing-plans
npx skills add obra/superpowers/verification-before-completion
npx skills add obra/superpowers/systematic-debugging
```

**Notas sobre las skills:**
- Se instalan en la carpeta `.skills/` dentro de la raíz del proyecto
- Son específicas por proyecto (no se comparten entre proyectos)
- Para verificar qué skills tiene un repo: `npx skills add repo/nombre --list`
- Para actualizar: reinstalar el mismo comando sobreescribe la versión anterior
- Agregar `.skills/` al `.gitignore` si no quieres subirlas al repo (ocupan espacio)

---

## 13. CONSIDERACIONES PARA VERSIÓN PRO (FUTURO)

Elementos a considerar para la versión pagada:

- **Multi-diagnóstico**: La empresa puede hacer múltiples diagnósticos en el tiempo (comparativa evolutiva)
- **Benchmark industrial**: Comparar con empresas del mismo sector
- **Dashboard avanzado**: Gráficos interactivos, drill-down por área
- **Diagnóstico en profundidad**: Más preguntas, entrevistas simuladas con IA
- **Integración con herramientas**: Conexión directa con CRM/ERP para datos reales
- **White-label**: Personalización con marca del cliente
- **Soporte multiidioma**: Inglés, español, portugués
- **API abierta**: Para integrar con otros sistemas
- **Planes de acción automatizados**: Con tracking de implementación
- **Consultoría IA**: Chat en vivo con IA para resolver dudas post-diagnóstico
- **Pagos**: Stripe o similar
- **Límites FREE vs PRO**: Número de empleados, áreas evaluables, profundidad del reporte

---

## 14. MVP — ORDEN DE DESARROLLO SUGERIDO

### Sprint 1 (Semana 1-2): Fundamentos
1. Setup monorepo: `backend/` (Express) + `frontend/` (React Vite) + Dockerfile
2. Crear repo en GitHub, configurar branches (main/develop)
3. Configurar Supabase: proyecto, tablas, Auth con Google OAuth, Storage bucket
4. Autenticación Google OAuth (Gmail + Workspace)
5. Landing page + página de términos y privacidad
6. Deploy inicial en Easypanel (App from GitHub) con dominio `diagnostico-ia.tribiu.com`

### Sprint 2 (Semana 3-4): Flujo CEO
5. Formulario de registro de empresa (Input #1)
6. Generación de código único
7. Integración IA para pre-diagnóstico empresa
8. Dashboard CEO (básico)
9. Flujo n8n para email con código

### Sprint 3 (Semana 5-6): Flujo Empleado
10. Login empleado (Google + código)
11. Formulario perfil empleado (Input #2)
12. Integración IA para perfil empleado
13. Generación de preguntas personalizadas
14. Interfaz de diagnóstico (Likert + cerradas + abiertas)

### Sprint 4 (Semana 7-8): Diagnóstico Final
15. Cierre automático por expiración
16. Generación de diagnóstico final con IA
17. Generación de PDF
18. Dashboard CEO completo con avance y descarga
19. Panel Superadmin

### Sprint 5 (Semana 9-10): Pulido y Deploy Final
20. Testing end-to-end (flujo completo CEO → Empleado → Diagnóstico)
21. Optimización de prompts con datos reales de prueba
22. Configurar DNS: `diagnostico-ia.tribiu.com` → IP del VPS
23. SSL automático vía Easypanel (Let's Encrypt)
24. Configurar flujos n8n en instancia existente
25. QA final con empresa piloto

---

*Documento generado como especificación base para el proyecto DiagnósticoIA v1 (FREE)*
