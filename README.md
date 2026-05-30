# 🐶 Tienda de Alimentos para Perritos

Aplicación web de 3 capas con pipeline CI/CD completo implementado en GitHub Actions.

![Pipeline](https://github.com/joralbornoz/ep2-tienda-perritos/actions/workflows/ci-cd.yml/badge.svg)

---

## 🏗️ Arquitectura

```text
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Frontend  │────▶│   Backend   │────▶│   MySQL DB  │
│ HTML + Nginx│     │ Node.js +   │     │             │
│  Port 8080  │     │ Express     │     │  Port 3306  │
│             │     │  Port 3001  │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
```

---

## 🚀 Pipeline CI/CD

Cada `push` a `main` ejecuta automáticamente las siguientes etapas:

Push a main
│
▼
┌─────────────────┐
│ 🧪 Pruebas      │  Jest · 11 tests · cobertura 90%
│ Unitarias       │
└────────┬────────┘
│
▼
┌─────────────────┐
│ 🔍 SonarCloud   │  Análisis de calidad y seguridad
│ Analysis        │  Quality Gate bloquea si falla 🚨
└────────┬────────┘
│
▼
┌─────────────────┐
│ 🐳 Build Docker │  Construye imágenes backend y frontend
│ Images          │  Etiquetadas con SHA del commit
└────────┬────────┘
│
▼
┌─────────────────┐
│ 🚀 Deploy       │  Docker Compose levanta los 3 servicios
│ Entorno         │  Health checks validan el despliegue
│ Simulado        │
└─────────────────┘

### Etapas del pipeline

| Etapa | Herramienta | Descripción |
|-------|-------------|-------------|
| 🧪 Pruebas Unitarias | Jest + Supertest | 11 tests con 90% de cobertura de código |
| 🔍 Análisis de Seguridad | SonarCloud | Detecta vulnerabilidades y code smells |
| 🚨 Quality Gate | SonarCloud | Bloquea el pipeline si falla el análisis |
| 🐳 Build Docker | Docker | Construye imágenes optimizadas con node:18-alpine |
| 🚀 Deploy | Docker Compose | Despliega y valida los 3 servicios automáticamente |
| 🔒 Dependencias | Dependabot | Revisa dependencias desactualizadas semanalmente |

---

## 🔍 Trazabilidad

Cada imagen Docker es etiquetada con el SHA del commit de GitHub:

```bash
tienda-perritos-backend:f159809f97ed914dac520f3f58616b9eb0dddfe2
```

Esto permite rastrear exactamente qué versión del código está desplegada en producción, garantizando trazabilidad completa desde el desarrollo hasta el despliegue.

---

## 🔒 Seguridad y Calidad

- **SonarCloud** analiza el código en cada push detectando vulnerabilidades, bugs y code smells
- **Quality Gate** bloquea automáticamente el pipeline si se detectan problemas críticos
- **Dependabot** revisa semanalmente las dependencias npm y Docker en busca de actualizaciones de seguridad
- **Dockerfile** usa `npm ci --ignore-scripts` para evitar ejecución de scripts maliciosos durante la instalación

---

## 🐳 Orquestación de Contenedores

La aplicación usa **Docker Compose** para orquestar 3 servicios:

```yaml
Servicios:
  ├── frontend   → Nginx sirviendo HTML/JS  (puerto 8080)
  ├── backend    → Node.js + Express API    (puerto 3001)
  └── db         → MySQL                   (puerto 3306)
```

Los servicios están configurados con:
- **Dependencias entre servicios** (`depends_on`) para orden de arranque correcto
- **Red interna** para comunicación segura entre contenedores
- **Volumen persistente** para los datos de MySQL
- **Variables de entorno** para configuración sin hardcodear credenciales

---

## 🖥️ Self-Hosted Runner — Deploy Local Automático

El pipeline utiliza un **self-hosted runner** instalado en la máquina local, lo que permite que cada `push` a `main` despliegue automáticamente los cambios en `http://localhost:8080`.

### ¿Cómo funciona?

```
Push a GitHub → GitHub Actions detecta el cambio →
Runner local recibe el job → Docker reconstruye las imágenes →
Contenedores actualizados → Cambios visibles en localhost:8080
```

### Configurar el Self-Hosted Runner (solo la primera vez)

Abre **PowerShell como Administrador** y ejecuta:

```powershell
# 1. Crear carpeta del runner
cd C:\
mkdir actions-runner
cd actions-runner

# 2. Descargar el runner
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.334.0/actions-runner-win-x64-2.334.0.zip -OutFile actions-runner-win-x64-2.334.0.zip

# 3. Extraer
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::ExtractToDirectory("$PWD/actions-runner-win-x64-2.334.0.zip", "$PWD")

# 4. Configurar con el token del repositorio
./config.cmd --url https://github.com/joralbornoz/ep2-tienda-perritos --token TU_TOKEN

# 5. Habilitar ejecución de scripts
Set-ExecutionPolicy RemoteSigned -Scope LocalMachine -Force
```

> El token se obtiene en GitHub → Settings → Actions → Runners → New self-hosted runner

### Iniciar el Runner

Cada vez que quieras que el deploy automático funcione, abre **PowerShell como Administrador**:

```powershell
cd C:\actions-runner
./run.cmd
```

Deberías ver:
```
√ Connected to GitHub
Listening for Jobs
```

### Ver cambios en tiempo real

1. Asegúrate que el runner está activo (`Listening for Jobs`)
2. Haz cualquier cambio en el código y súbelo:

```bash
git add .
git commit -m "descripcion del cambio"
git push origin main
```

3. El pipeline correrá automáticamente (~2 minutos)
4. Abre `http://localhost:8080` — los cambios estarán desplegados

### Verificar contenedores activos

```powershell
docker compose ps
```

```
tienda-frontend   running   0.0.0.0:8080->80/tcp
tienda-backend    running   0.0.0.0:3001->3001/tcp
tienda-db         running   0.0.0.0:3306->3306/tcp
```

---

## 📁 Estructura del Proyecto

```text
ep2-tienda-perritos/
├── .github/
│   ├── workflows/
│   │   └── ci-cd.yml          # Pipeline CI/CD
│   └── dependabot.yml         # Escaneo de dependencias
├── backend/
│   ├── tests/
│   │   └── server.test.js     # Tests unitarios (Jest)
│   ├── Dockerfile             # Imagen optimizada node:18-alpine
│   ├── server.js              # API REST Express
│   └── package.json
├── frontend/
│   ├── Dockerfile
│   ├── index.html
│   └── app.js
├── db/
│   └── init.sql               # Script de inicialización MySQL
├── sonar-project.properties   # Configuración SonarCloud
├── docker-compose.yml         # Orquestación de contenedores
└── README.md
```

---

## 🛠️ Cómo ejecutar localmente

### Requisitos
- Docker Desktop instalado
- Self-hosted runner activo (para deploy automático)

### Pasos

```bash
# 1. Clonar el repositorio
git clone https://github.com/joralbornoz/ep2-tienda-perritos.git
cd ep2-tienda-perritos

# 2. Levantar todos los servicios
docker compose up -d --build

# 3. Verificar que están corriendo
docker compose ps
```

### URLs disponibles

| Servicio | URL |
|----------|-----|
| Frontend | http://localhost:8080 |
| Backend API | http://localhost:3001/api/productos |
| Health Check | http://localhost:3001/api/health |

### Detener los servicios

```bash
docker compose down
```

---

## 🧪 Ejecutar Tests

```bash
cd backend
npm install
npm test
```

Resultado esperado:
Tests:       11 passed, 11 total
Coverage:    90.47%

---

## 🔌 Endpoints de la API

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/health` | Estado del servicio |
| GET | `/api/productos` | Listar todos los productos |
| GET | `/api/productos/:id` | Obtener producto por ID |
| POST | `/api/productos` | Crear nuevo producto |
| PUT | `/api/productos/:id` | Actualizar producto |
| DELETE | `/api/productos/:id` | Eliminar producto |

---

## 🤖 Uso de Inteligencia Artificial

Durante el desarrollo de este proyecto se utilizó **Claude (Anthropic)** como herramienta de apoyo para:
- Generación de estructura base del pipeline CI/CD
- Configuración de SonarCloud con GitHub Actions
- Generación de tests unitarios con Jest

Todos los contenidos fueron revisados, validados y adaptados por el equipo según los requerimientos del proyecto.

Referencia: https://bibliotecas.duoc.cl/ia


