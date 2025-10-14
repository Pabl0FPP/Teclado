# Teclado - Aplicación de Práctica de Mecanografía

**Proyecto Original:** [Issblann/Teclado](https://github.com/Issblann)

**Integración CI/CD:**
- Santiago Valencia García - A00395902
- Pablo Fernando Pineda Patiño - A00395831

Esta aplicación web interactiva permite mejorar las habilidades de mecanografía mediante ejercicios de tecleo guiados. El proyecto original fue desarrollado por Issblann y se le integró análisis de calidad de código con SonarQube a través de un pipeline automatizado en Jenkins como parte de un ejercicio académico de DevOps.

## Descripción General

Teclado es una aplicación web estática desarrollada en HTML, CSS y JavaScript puro (Vanilla JS) que simula un teclado virtual en pantalla. La aplicación resalta teclas de forma aleatoria y valida que el usuario las presione correctamente, proporcionando retroalimentación visual inmediata mediante animaciones CSS.

Este fork del proyecto original incorpora infraestructura de integración continua y análisis estático de código, demostrando la implementación de prácticas DevOps sobre una aplicación web existente.

## Características de la Aplicación

**Interfaz Visual Interactiva**  
La aplicación renderiza un teclado completo en pantalla con representación visual de todas las teclas estándar (ESC, números, letras, modificadores). Cada tecla está estilizada según el dedo recomendado para pulsarla (meñique, anular, medio, índice) usando clases CSS diferenciadas.

**Lógica de Juego**  
El sistema selecciona teclas aleatoriamente y las resalta con la clase `selected`. Cuando el usuario presiona la tecla física correcta, el evento `keydown` captura la entrada, aplica una animación de pulsación (`hit`), valida la coincidencia y selecciona una nueva tecla objetivo.

**Retroalimentación Visual**  
Se utiliza la API de animaciones CSS (`animationend` event listener) para aplicar y remover clases dinámicamente, creando efectos visuales fluidos que indican tanto teclas objetivo como teclas presionadas.

**Responsividad**  
El diseño CSS (compilado desde SCSS) adapta el layout del teclado virtual a diferentes resoluciones de pantalla, manteniendo proporciones y legibilidad.

## Contribuciones al Proyecto Original

Este repositorio es un fork del proyecto original de Issblann con las siguientes adiciones específicas para implementación de CI/CD:

**Archivo Jenkinsfile**  
Pipeline completo de integración continua que automatiza el análisis de calidad de código con SonarQube. Este es el único archivo añadido al proyecto original.

**Integración con Infraestructura DevOps**  
Configuración del proyecto en Jenkins mediante Job DSL (definido en el servidor Jenkins) para ejecutar automáticamente el pipeline en cada cambio del código.

**Configuración SonarQube**  
Parámetros de análisis estático adaptados a la estructura del proyecto (exclusiones de CSS, análisis de JavaScript, project key y name).

El resto de archivos (`index.html`, `script.js`, `css/*`) pertenecen íntegramente al trabajo original de Issblann y no han sido modificados.

## Arquitectura del Proyecto Original

### Componentes Frontend

**index.html**  
Estructura HTML semántica que define:
- Encabezados instructivos ("OJOS EN LA PANTALLA" / "MANOS EN EL TECLADO")
- Contenedor `.keyboard` con listas desordenadas representando filas del teclado
- Cada tecla como elemento `<li>` con ID único (correspondiente al valor de la tecla) y clases para estilización
- Referencias a hoja de estilos CSS y script JavaScript

**script.js**  
Lógica de aplicación implementada en JavaScript puro:
- `getRandomNumber(min, max)`: generador de números aleatorios
- `getRandomKey()`: selector aleatorio de elementos del DOM
- `targetRandomKey()`: aplica clase `selected` a tecla aleatoria
- Event listener `keydown`: captura eventos de teclado, valida coincidencias, aplica/remueve clases, controla flujo del juego

**css/style.css**  
Hoja de estilos compilada desde SCSS que define:
- Variables de color y tipografía
- Layout del teclado (flexbox/grid)
- Clases de estado (`.selected`, `.hit`)
- Animaciones de transición
- Diferenciación visual por dedo (`.pinky`, `.ring`, `.middle`, `.pointer1st`, `.pointer2nd`)

**css/style.scss**  
Código fuente SASS con variables, mixins, nesting y estructura modular para facilitar mantenimiento de estilos.

## Integración CI/CD (Contribución Académica)

La principal contribución a este proyecto es la implementación de un pipeline de integración continua y análisis de calidad de código, demostrando la aplicación de prácticas DevOps en un proyecto web existente.

### Infraestructura Implementada

El proyecto se integra con un stack completo de CI/CD desplegado en Azure:

- **Infraestructura:** Máquinas virtuales aprovisionadas con Terraform
- **Orquestación:** Servicios Docker desplegados con Ansible
- **Jenkins:** Servidor de integración continua con configuración declarativa (JCasC)
- **SonarQube:** Plataforma de análisis estático de código
- **PostgreSQL:** Base de datos para SonarQube

Esta infraestructura está documentada en los repositorios complementarios `terraform_for_each_vm` y `ansible-pipeline`.

## Pipeline de Jenkins

El archivo `Jenkinsfile` define un pipeline declarativo con las siguientes etapas:

### Variables de Entorno

**SONAR_HOST_URL**  
URL del servidor SonarQube configurada globalmente en Jenkins mediante JCasC (Jenkins Configuration as Code). Apunta a la instancia desplegada en Azure.

**SONAR_PROJECT_KEY**  
Identificador único del proyecto en SonarQube (`teclado-project`).

**SONAR_PROJECT_NAME**  
Nombre descriptivo del proyecto mostrado en el dashboard de SonarQube.

### Etapas del Pipeline

**Stage: Checkout**  
Clona el repositorio de código fuente desde el sistema de control de versiones (Git). Utiliza la configuración SCM (Source Code Management) definida automáticamente por Jenkins.

**Stage: Verificar Herramientas**  
Valida la disponibilidad de herramientas necesarias para el análisis:
- Node.js (versión 18.20.2 instalada en la imagen Docker de Jenkins)
- NPM (gestor de paquetes de Node.js)
- SonarScanner CLI (versión 5.0.1 instalada directamente en la imagen)

Ejecuta comandos `--version` para cada herramienta y muestra las versiones en los logs del build.

**Stage: Análisis de Código con SonarQube**  
Ejecuta análisis estático de código utilizando SonarScanner CLI directamente (sin plugin de Jenkins):

- Recupera token de autenticación desde credenciales de Jenkins (`SONAR_TOKEN`)
- Ejecuta `sonar-scanner` con parámetros:
  - `-Dsonar.projectKey`: identificador del proyecto
  - `-Dsonar.projectName`: nombre descriptivo
  - `-Dsonar.sources=.`: analizar todo el directorio raíz
  - `-Dsonar.exclusions=css/**,**/*.css`: excluir archivos CSS compilados
  - `-Dsonar.javascript.file.suffixes=.js`: analizar solo archivos JavaScript
  - `-Dsonar.host.url`: URL del servidor SonarQube
  - `-Dsonar.token`: credencial de autenticación

El análisis detecta code smells, bugs, vulnerabilidades de seguridad, duplicaciones y métricas de complejidad.

**Stage: Validación de Archivos**  
Verifica la integridad estructural del proyecto:
- Lista todos los archivos del workspace
- Valida existencia de archivos críticos (`index.html`, `script.js`)
- Verifica presencia de directorio `css/`
- Falla el pipeline si algún archivo crítico falta

### Secciones Post-Ejecución

**post: success**  
Se ejecuta cuando el pipeline completa exitosamente. Muestra mensaje de éxito y proporciona URL directa al dashboard de SonarQube para revisar resultados del análisis.

**post: failure**  
Se ejecuta cuando alguna etapa falla. Indica al usuario revisar logs para diagnóstico.

**post: always**  
Se ejecuta siempre independientemente del resultado. Registra finalización del pipeline.

## Integración con SonarQube

El proyecto está configurado para enviar métricas de calidad a SonarQube automáticamente en cada ejecución del pipeline.

### Configuración del Análisis

**Exclusiones**  
Se excluyen archivos CSS compilados del análisis para evitar ruido en las métricas, ya que estos archivos son generados automáticamente desde SCSS.

**Lenguajes Analizados**  
SonarQube analiza archivos JavaScript (`.js`) aplicando reglas específicas del lenguaje para detectar:
- Variables no utilizadas
- Funciones sin uso
- Complejidad ciclomática excesiva
- Patrones de código problemáticos
- Vulnerabilidades de seguridad (XSS, injection, etc.)

### Resultados de Calidad

El proyecto ha obtenido Quality Gate PASSED con las siguientes métricas:

- **Bugs:** 0
- **Vulnerabilidades:** 0
- **Code Smells:** 0
- **Duplicaciones:** 0%
- **Cobertura:** N/A (sin tests unitarios implementados)
- **Ratings:**
  - Reliability: A
  - Security: A
  - Maintainability: A

Estos resultados indican código limpio, bien estructurado y sin problemas críticos detectados por el análisis estático.

## Estructura del Proyecto

```
Teclado/
├── index.html          # Estructura HTML principal de la aplicación
├── script.js           # Lógica de juego y manejo de eventos
├── Jenkinsfile         # Definición del pipeline CI/CD
├── css/
│   ├── style.css       # Hoja de estilos compilada (CSS)
│   ├── style.scss      # Código fuente de estilos (SASS)
│   └── style.css.map   # Source map para debugging de estilos
└── .git/               # Control de versiones Git
```

## Flujo de Trabajo CI/CD

El flujo completo desde commit hasta análisis de calidad es:

1. **Desarrollo Local:** El desarrollador modifica código HTML/JS/SCSS
2. **Commit & Push:** Los cambios se suben al repositorio Git en GitHub
3. **Trigger Automático:** Jenkins detecta el push mediante webhook o polling
4. **Ejecución del Pipeline:**
   - Clona el repositorio
   - Verifica herramientas disponibles
   - Ejecuta análisis SonarQube
   - Valida estructura de archivos
5. **Publicación de Resultados:** Métricas se envían a SonarQube
6. **Dashboard:** Los desarrolladores revisan Quality Gate y métricas en interfaz web de SonarQube

## Configuración de Jenkins

La integración con Jenkins se realiza mediante Job DSL definido en el archivo `casc.yaml` del servidor Jenkins. El job se configura con:

- **Tipo:** Pipeline
- **SCM:** Git (repositorio GitHub Pabl0FPP/Teclado)
- **Script Path:** `Jenkinsfile` en la raíz del repositorio
- **Triggers:** Polling SCM o webhooks de GitHub

Las credenciales de SonarQube (`SONAR_TOKEN`) se inyectan automáticamente desde la configuración global de Jenkins, evitando hardcodear secretos en el código.

## Instalación y Ejecución Local

### Prerequisitos

- Navegador web moderno (Chrome, Firefox, Edge, Safari)
- Servidor web estático (opcional: Python, Node.js http-server, Live Server de VS Code)

### Ejecución Directa

Para entornos de desarrollo simple, abrir `index.html` directamente en el navegador funciona para la funcionalidad básica, aunque puede haber restricciones CORS con algunos recursos.

### Servidor Local con Python

```bash
# Python 3
python -m http.server 8000

# Abrir en navegador: http://localhost:8000
```

### Servidor Local con Node.js

```bash
# Instalar http-server globalmente
npm install -g http-server

# Ejecutar servidor
http-server -p 8000

# Abrir en navegador: http://localhost:8000
```

### Desarrollo con Live Server (VS Code)

Instalar extensión Live Server en VS Code y hacer clic derecho en `index.html` → "Open with Live Server". El servidor recarga automáticamente al detectar cambios.

## Compilación de Estilos SASS

Si se modifican los estilos en `style.scss`, es necesario recompilar a CSS:

```bash
# Instalar SASS
npm install -g sass

# Compilar SCSS a CSS
sass css/style.scss css/style.css

# Compilar en modo watch (recompilación automática)
sass --watch css/style.scss:css/style.css
```

## Mejoras Futuras

**Testing Automatizado**  
Implementar tests unitarios con Jest para validar funciones JavaScript (`getRandomNumber`, `getRandomKey`) y tests end-to-end con Cypress para validar flujo de usuario.

**Cobertura de Código**  
Integrar reportes de cobertura (Istanbul/NYC) en el pipeline de Jenkins y enviarlos a SonarQube para visibilidad de áreas no testeadas.

**Métricas de Usuario**  
Añadir contador de teclas correctas/incorrectas, temporizador, cálculo de palabras por minuto (WPM) y precisión porcentual.

**Progresión de Dificultad**  
Implementar niveles de dificultad con diferentes conjuntos de teclas (solo letras, letras + números, teclado completo, combinaciones con Shift).

**Persistencia de Datos**  
Utilizar LocalStorage para guardar estadísticas de práctica, progreso del usuario y mejores puntuaciones.

**Accesibilidad**  
Mejorar soporte para lectores de pantalla (ARIA labels), navegación por teclado y contraste de colores según WCAG 2.1.

**Internacionalización**  
Soporte para múltiples layouts de teclado (QWERTY, AZERTY, QWERTZ, Dvorak) y localización de textos.

## Seguridad y Buenas Prácticas

**Validación de Entrada**  
El código valida eventos de teclado contra elementos DOM existentes (`getElementById`) antes de aplicar clases, previniendo errores de referencia nula.

**No Dependencias Externas**  
La aplicación utiliza JavaScript puro sin frameworks ni librerías externas, reduciendo superficie de ataque y simplificando auditorías de seguridad.

**Content Security Policy**  
Para despliegue en producción se recomienda configurar CSP headers que permitan solo scripts inline propios y recursos del mismo origen.

**HTTPS en Producción**  
Si se despliega públicamente, utilizar HTTPS para proteger integridad de recursos y prevenir ataques man-in-the-middle.

## Recursos de Aprendizaje

Este proyecto demuestra conceptos fundamentales de desarrollo web y DevOps:

- **DOM Manipulation:** Selección, modificación y eventos del Document Object Model
- **Event Handling:** Listeners de eventos de teclado y animaciones
- **CSS Animations:** Transiciones y keyframes para retroalimentación visual
- **SASS/SCSS:** Preprocesadores CSS con variables y nesting
- **CI/CD Pipelines:** Automatización de análisis de calidad con Jenkins
- **Static Code Analysis:** Integración de SonarQube para métricas de código
- **Infrastructure as Code:** Jenkins CasC para configuración declarativa

## Estado Actual del Proyecto

**Aplicación Original:** Completamente funcional, código desarrollado por Issblann

**Infraestructura CI/CD:** Operativa y desplegada
- **Pipeline:** Ejecutado exitosamente (build #4 último exitoso)
- **SonarQube:** Quality Gate PASSED, sin issues detectados
- **Jenkins:** Servidor en Azure VM (20.12.193.55:80)
- **SonarQube Dashboard:** Disponible en puerto 9000

El código del proyecto original está preservado sin modificaciones. La única adición es el archivo `Jenkinsfile` que habilita la integración con el stack DevOps desplegado mediante Terraform y Ansible.

## Créditos

**Desarrollo de la Aplicación:** [Issblann](https://github.com/Issblann) - Autor original del proyecto Teclado

**Integración CI/CD:** Santiago Valencia García y Pablo Fernando Pineda Patiño - Implementación de pipeline Jenkins, análisis SonarQube e infraestructura DevOps como ejercicio académico
