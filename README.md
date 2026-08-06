Pvox - Plataforma Web de Red Social y Arquitectura de Sistema

Demostración en vivo: https://pvox.42web.io/views/pages/login.php

Nota sobre el repositorio: Este repositorio público funciona como documentación técnica, caso de estudio y demostración de arquitectura del sistema. El código fuente de la aplicación se gestiona en un repositorio privado por razones de seguridad.

Resumen del Proyecto
Pvox es una plataforma de red social orientada a la libre expresión y la interacción asíncrona en tiempo real. El sistema fue diseñado bajo una arquitectura modular en PHP sin el uso de frameworks pesados, aplicando un patrón MVC artesanal, comunicación vía API REST con consumo asíncrono (JSON) y un pipeline de procesamiento distribuido para archivos multimedia.

Eslogan
•	"Comparte lo que piensas, conecta con quien quieras."
•	"Tu espacio, tu voz, tu libertad. Creemos en la expresión auténtica y sin represiones: di lo que piensas y sé quien eres en una comunidad donde el único límite es el respeto mutuo."

Arquitectura de Sistema y Flujo de Datos
Para superar las limitaciones físicas de alojamiento en servidores de producción (como los límites de carga de 20 MB en hosting compartido), el sistema delega las tareas pesadas de cómputo al cliente y a servicios desacoplados en el Edge.
[ Navegador del Usuario ]
       │
       ├─► Compresión de Imagen (JS Canvas) ──► Genera WebP
       ├─► Compresión de Video (FFmpeg Web Worker en Netlify) ──► Reducción a 720p
       │
       ▼ (Petición HTTP Asíncrona / JSON Payload)
[ Servidor Web & API REST - PHP ]
       │
       ├─► Validación de Seguridad (Geobloqueo, Hashing BCRYPT, Sanitización PDO)
       ├─► Persistencia de Datos (Base de Datos MySQL)
       ├─► Gestión de Archivos (Almacenamiento Local Estructurado por Fecha)
       └─► Tareas Automatizadas (Cron Jobs / Procesamiento de DMCA y Strikes)

Funcionalidades del Sistema
1. Experiencia del Usuario Final
•	Autenticación y Seguridad: Registro, inicio de sesión, recuperación de contraseña por correo electrónico con plantillas HTML parametrizadas y validación de sesiones activas.
•	Feed Asíncrono e Interacciones: Carga de publicaciones sin recargar la página mediante Fetch API, sistema de reaciones, comentarios dinámicos, gestión de favoritos y publicaciones compartidas.
•	Pipeline Multimedia en el Cliente:
o	Conversión automática de imágenes a formato optimizado WebP utilizando la API de Canvas en JavaScript antes del envío.
o	Procesamiento de video mediante Web Workers ejecutando FFmpeg (alojado en Netlify) para transcodificar medios pesados a 720p localmente, evitando saturar el ancho de banda del servidor.
•	Sistema de Seguimiento y Mensajería: Permite seguir/dejar de seguir usuarios, búsqueda filtrada de perfiles y mensajería directa entre cuentas.
•	Reportes de Errores (Bug Tracking): Módulo para que los usuarios envíen reportes de fallos del sistema especificando título, descripción, nivel de prioridad, tipo de interfaz y comportamiento esperado.

2. Gobernanza, Moderación y Administración
El sistema integra un esquema de control de acceso basado en roles (RBAC) con una jerarquía de 4 niveles:
Nivel	Rol	Permisos y Capacidades
0	Usuario	Uso estándar de la plataforma, publicación de contenido, envío de reportes y denuncias de copyright.
1	Moderador Nivel 1	Capacidad de ocultar publicaciones de forma inmediata al reportarlas, omitiendo el umbral matemático del algoritmo automático.
2	Moderador Nivel 2	Mismas funciones de Nivel 1 + capacidad de proponer la promoción de usuarios a rangos administrativos mediante votación.
3	Administrador Nivel 3	Usuario raíz único. Control total sobre el estado del sistema, gestión de rangos sin mediación de votación, configuración de avisos y activación de mantenimiento.
•	Algoritmo Matemático de Ocultación: Las publicaciones reportadas por usuarios estándar se procesan mediante un algoritmo que calcula la proporción de denuncias vs. interacciones. Si se supera el umbral establecido, la publicación se oculta automáticamente hasta ser revisada.
•	Sistema de Sanciones y Strikes: Acumular 5 infracciones (strikes) resulta en el bloqueo permanente de la dirección de correo electrónico. La plataforma incluye un flujo de apelación vía correo con dos oportunidades durante un periodo de 10 días laborables.
•	Mantenimiento y Bypass por Código: El Administrador Nivel 3 puede activar un bloqueo global del sistema, expulsando a las sesiones activas. El acceso durante el mantenimiento requiere introducir una clave dinámica configurada previamente.
•	Versionado Legal de Políticas: Control de versiones para Términos de Uso y Políticas de Privacidad. Si se incrementa la versión global, todos los usuarios son redirigidos a aceptar las nuevas condiciones antes de continuar navegando.


Stack Tecnológico
•	Frontend: HTML5, CSS3 (Estructura modular BEM/Custom Properties), JavaScript Vanilla (ES6+, Fetch API, Web Workers, Canvas API).
•	Backend: PHP 8.x (Programación Orientada a Objetos, Arquitectura API REST sin frameworks).
•	Base de Datos: MySQL 8.0, conexión mediante PHP Data Objects (PDO) con sentencias preparadas.
•	Procesamiento Distribuido: Worker de FFmpeg ejecutado en infraestructura Netlify.
•	Servicios de Correo: Integración de PHPMailer para el envío de notificaciones de seguridad, DMCA y autenticación.
•	Entorno de Desarrollo: Docker y Docker Compose para la estandarización del contenedor de aplicación y base de datos.

Estructura de Directorios
├── api/
│   ├── admin/             # Endpoints para moderación, copyright y promociones
│   ├── auth/              # Procesamiento de login, registro y recuperación
│   ├── cron/              # Tareas programadas de fondo (Strikes DMCA)
│   ├── message/           # Motor de mensajería interna
│   ├── posts/             # Creación, interacción, descarga y reportes
│   ├── search/            # Motor de búsqueda
│   ├── settings/          # Configuración de cuenta y seguridad
│   ├── support/           # Sistema de tickets y reporte de errores
│   ├── system/            # Monitor de estado mediante SSE
│   └── users/             # Gestión de perfiles y seguimiento
├── assets/
│   ├── css/               # Estilos divididos por componentes, páginas y módulos
│   ├── js/                # Controladores del cliente, workers y lógica AJAX
│   └── img/               # Recursos gráficos y favicons
├── config/                # Archivos de entorno, base de datos y geobloqueo
├── core/                  # Bootstrap, autenticación central, MailEngine y seguridad
│   ├── maintenance/       # Scripts de migración, instalación y limpieza
│   └── security/          # Filtros de red y geolocalización
├── storage/               # Archivos de sistema, perfiles y medios estructurados por fecha
├── vendor/                # Dependencias (PHPMailer)
└── views/                 # Vistas PHP, componentes modulares y plantillas de correo
    ├── components/
    ├── emails/
    ├── legal/
    ├── pages/
    └── partials/
    
Aspectos Destacados de Ingeniería
1. Offloading en el Cliente (Edge Offloading)
Para evitar fallos por tiempo de espera (timeout) o límites de carga en el servidor web, el procesamiento pesado de medios se ejecuta en el navegador antes de iniciar la transferencia HTTP. Los videos se procesan mediante un Web Worker que ejecuta un binario de FFmpeg compilado en WebAssembly y alojado externamente, garantizando que el servidor principal solo reciba archivos optimizados.

3. Monitoreo mediante Server-Sent Events (SSE)
El estado de la plataforma y del modo mantenimiento se verifica a través del archivo api/system/sse_status.php. En lugar de realizar peticiones HTTP iterativas (polling) que saturan la base de datos, el sistema mantiene una conexión unidireccional persistente basada en eventos SSE.
4. Seguridad e Integridad de Datos
•	Criterios de Conexión: Acceso a la base de datos centralizado mediante PDO, forzando la parametrización de variables (Prepared Statements) para la mitigación total de inyecciones SQL.
•	Protección de Credenciales: Contraseñas procesadas mediante la función nativa password_hash() con el algoritmo BCRYPT.
•	Filtrado de Red: Control de acceso y geobloqueo integrado en la capa core/security/geo_filter.php.
•	Mantenimiento de Almacenamiento: Módulo cleanup_zombies.php para la eliminación periódica de archivos multimedia huérfanos que no poseen registros asociados en la base de datos.

