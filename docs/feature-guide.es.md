# Resumen de funciones de Launchpad

Esta guía es un resumen conciso del kit inicial de Launchpad para compradores potenciales. Resume lo que está incluido, lo que requiere servicios externos o configuración, y lo que el kit no proporciona automáticamente. La guía completa de funciones y el código fuente del proyecto adquirido siguen siendo las referencias autorizadas de implementación.

## Qué incluye Launchpad

Launchpad es una aplicación SaaS modular construida sobre una base de código compartida. Sus módulos opcionales cubren facturación, chat de IA, organizaciones, publicación, soporte, comunidad, notificaciones, marketing por correo electrónico, analítica, referidos, publicidad, SEO, integraciones y almacenamiento privado de archivos. El asistente de configuración le permite elegir qué módulos opcionales conservar y un destino de despliegue. Los componentes centrales de cuenta, seguridad, configuración y base de datos compartida permanecen en su lugar.

En una extracción de Git, la configuración utiliza sparse-checkout, por lo que los archivos rastreados excluidos permanecen en el historial de Git. En un ZIP adquirido, los archivos de los módulos excluidos se eliminan del proyecto extraído; no pueden restaurarse desde el historial de Git porque el ZIP no contiene metadatos del repositorio. Esta es una configuración de una sola vez, no una herramienta de conmutación de módulos en el lugar. Realice una copia de seguridad del ZIP extraído antes de configurarlo si es posible que desee recuperar el código fuente excluido más adelante.

El asistente escribe los módulos seleccionados y las preferencias de despliegue en `launchpad.config.json`, y los secretos de los proveedores en el archivo `.env` ignorado por Git. Puede verificar las credenciales de los proveedores seleccionados cuando elige explícitamente **Probar conexión**. No crea cuentas de proveedor, no despliega infraestructura ni configura paneles de control externos. Aún debe completar y mantener esos servicios usted mismo.

El área de plugins expone un conjunto de componentes incluidos en la lista de permitidos; no es un mercado de paquetes arbitrario y no descarga ni ejecuta código de plugins cargados. El servicio opcional de retransmisión de eventos puede desplegarse por separado para el evento de actualización de perfil de cuenta actualmente soportado. No es un registro de servicios general, un bus de mensajes ni un sistema de confirmación por consumidor.

## Cuentas y seguridad

La autenticación admite inicio de sesión por correo electrónico e inicio de sesión opcional con Google y GitHub. En desarrollo, Mailpit puede recibir correos de inicio de sesión cuando no se ha configurado un proveedor de correo electrónico. Los entornos desplegados necesitan un proveedor de correo electrónico para el inicio de sesión por correo y el correo transaccional. Las páginas y API de administrador aplican roles en el servidor.

La autenticación multifactor opcional TOTP funciona con aplicaciones de autenticación compatibles e incluye códigos de recuperación de un solo uso. La configuración de seguridad incluye la gestión de sesiones y la actividad de auditoría. El kit no incluye inicio de sesión basado en contraseñas ni un flujo de recuperación de contraseña.

La lista de verificación de incorporación de la cuenta cubre la configuración del perfil, la seguridad de la cuenta y el panel de control; es opcional y puede revisarse.

## Facturación y acceso de pago

El módulo de facturación admite Stripe Checkout para suscripciones configuradas y compras únicas de por vida, además de la Stripe Customer Portal. La verificación opcional de compras de Apple App Store y Google Play puede proporcionar otra fuente de facturación cuando se configuran los productos, credenciales, cuentas de tienda e integración de cliente nativo requeridos. Las rutas de la tienda no están disponibles hasta que se proporcionen sus configuraciones; Stripe puede continuar operando de forma independiente.

El kit procesa eventos de pago y suscripción verificados, registra el historial de pagos y maneja reembolsos y disputas. No **protege** automáticamente cada función de su producto solo porque la facturación esté habilitada. `paymentBlocked` es una señal de facturación, no un muro de pago universal. Cada función de pago debe aplicar el acceso en el servidor; ocultar un botón en el navegador no es suficiente. Decida qué funciones son de pago y qué pueden hacer aún los clientes restringidos, luego implemente y pruebe esas reglas. El kit tampoco incluye un flujo de trabajo de administrador listo para usar para verificar pagos realizados fuera de Stripe.

El correo de facturación utiliza el proveedor de correo configurado, pero la facturación no requiere correo para funcionar. La entrega de correos puede retrasarse, fallar o duplicarse después de una respuesta de red incierta; no es un sistema de entrega exactamente una vez.

## Correo electrónico, lista de espera y marketing

El transporte de correo compartido admite Resend, SendGrid y SMTP. Se utiliza para autenticación, avisos de facturación, notificaciones, boletines y mensajes de marketing. Las identidades del remitente y las credenciales del proveedor deben configurarse. La comprobación de SMTP del asistente de configuración autentica sin enviar un mensaje.

La lista de espera de acceso anticipado localizada requiere consentimiento explícito. Sus estados se gestionan manualmente: invitar o marcar a alguien como unido no crea una cuenta, no envía una invitación ni otorga acceso. El consentimiento de la lista de espera es separado de la suscripción al boletín.

El boletín admite registros basados en consentimiento, enlaces de cancelación de suscripción y campañas de texto plano creadas por el administrador. La automatización de marketing admite secuencias cortas de correos para nuevos suscriptores del boletín. Estas herramientas no son un editor visual de correo, un sistema de segmentación de audiencia ni un servicio de cumplimiento/legal. No inscriben automáticamente entradas de la lista de espera ni envían mensajes a través de notificaciones push y webhooks. Las campañas programadas y la automatización requieren un programador externo además de un proveedor de correo configurado.

## Asistente de IA y organizaciones

El asistente opcional del panel de control se conecta a una API de texto compatible con OpenAI configurada. Almacena conversaciones en la base de datos de la aplicación, aplica límites de solicitud de cuenta y admite conversaciones personales y de organización. No incluye un modelo, no ejecuta herramientas ni toma acciones autónomas. Los prompts y respuestas se envían al proveedor configurado y son almacenados por la aplicación; configure la configuración de privacidad y retención del proveedor y divulgue este procesamiento a los usuarios.

Las organizaciones proporcionan espacios de trabajo compartidos, roles de miembros e invitaciones. La autorización se comprueba contra la membresía de la organización, pero esta es una tenencia de base de datos compartida, no una base de datos o esquema separado para cada cliente. La facturación, la configuración de la cuenta, el soporte, las notificaciones, la analítica, los boletines y el contenido de publicación global no están automáticamente acotados a la organización.

## Publicación, soporte y comunidad

Módulos separados proporcionan un blog Markdown localizado, un feed de actualizaciones de versión y páginas CMS. Los administradores gestionan borradores y contenido publicado. Las páginas públicas incluyen metadatos localizados y soporte de sitemap. Las herramientas de publicación no incluyen publicación programada, traducción automática, carga de imágenes ni un sistema de moderación de comentarios.

El módulo de soporte proporciona tickets de cliente y respuestas de administrador. El módulo de comunidad proporciona un foro de discusión. El directorio curado permite a los administradores gestionar entradas públicas. Estas son funciones iniciales en lugar de integraciones con un escritorio de ayuda alojado o una red social completa: los adjuntos, la automatización de asignación/SLA, las reacciones del foro, las respuestas aceptadas y los perfiles de usuario públicos no están incluidos.

## Notificaciones y datos del cliente

El panel de control tiene una bandeja de entrada de notificaciones persistente y canales de entrega opcionales de correo electrónico, Web Push y webhook firmado. Las notificaciones push requieren claves VAPID y suscripción del navegador; los consumidores de webhook deben verificar firmas y deduplicar eventos. Los reintentos automáticos para notificaciones, marketing y limpieza de almacenamiento necesitan trabajos programados externamente.

La configuración de la cuenta proporciona una exportación JSON de los datos de cuenta soportados y un enlace separado a la gestión de facturación. La exportación excluye credenciales, sesiones, secretos de firma y otros tokens sensibles. Es una descarga en flujo, no una copia de seguridad completa de la base de datos ni un servicio de archivo en segundo plano.

## Analítica, localización y personalización

El panel de control incluye vistas de cuenta y facturación. La analítica de producto opcional recopila eventos con control de consentimiento y proporciona informes administrativos; no son un reemplazo para una plataforma de analítica de terceros. La atribución, el seguimiento de marketing y las cookies de referidos pueden tener obligaciones de consentimiento en su despliegue. Launchpad no incluye una plataforma de gestión de consentimiento.

La interfaz admite 18 localizaciones: inglés, árabe, bengalí, alemán, español, francés, hindi, indonesio, italiano, japonés, coreano, neerlandés, polaco, portugués, ruso, turco, vietnamita y chino. El inglés es el catálogo de respaldo y el idioma de respaldo del asistente de configuración. Revise las traducciones y el texto legal para su audiencia antes del lanzamiento.

El constructor de producto edita un conjunto de secciones de aplicación en la lista de permitidos, la navegación, el texto localizado y la configuración de marca. No genera código arbitrario, no crea rutas de aplicación ni cambia la lógica de negocio. El tema, el movimiento y otras preferencias de interfaz personal se almacenan en el navegador, no se sincronizan entre dispositivos.

## Almacenamiento, referidos, publicidad y SEO

El almacenamiento privado de archivos opcional utiliza almacenamiento de objetos compatible con S3. La aplicación valida tipos y tamaños de archivos, utiliza URLs de carga/descarga de vida corta y comprueba la propiedad del archivo. Configure un cubo privado y las credenciales requeridas. El módulo no escanea archivos en busca de malware, y la eliminación de la cuenta no elimina automáticamente los objetos remotos.

El módulo de referidos rastrea enlaces de socios y comisiones derivadas de Stripe. Los pagos se realizan fuera de Launchpad. No calcula comisiones para compras de App Store o Google Play. La atribución de referidos utiliza una cookie de 30 días; Launchpad no tiene un gestor de consentimiento integrado, por lo que evalúe e implemente los requisitos de consentimiento aplicables.

Las herramientas de publicidad organizan información de campañas y cuentas e importan informes desde CSV. No se conectan a redes publicitarias para crear o gestionar campañas en vivo. Las funciones de SEO proporcionan metadatos localizados, sitemaps, reglas de robots, datos estructurados y una auditoría de la base de datos de la aplicación; no son un rastreador web, un rastreador de clasificación, una herramienta de investigación de palabras clave ni un gestor de redirecciones.

El catálogo de integraciones informa si la configuración esperada está presente. No demuestra que las credenciales funcionen, no crea cuentas de proveedor ni garantiza la entrega.

## Despliegue y responsabilidades operativas

Los despliegues requieren PostgreSQL y variables de entorno apropiadas, incluidas las conexiones de base de datos y `AUTH_SECRET`. Configure correo, Stripe, facturación de tienda, almacenamiento de objetos, IA, push y trabajos programados solo para las funciones que habilite. Las migraciones de producción deben aplicarse utilizando el proceso de despliegue documentado; mantenga los entornos de vista previa en una base de datos aislada.

La aplicación incluye validación del lado del servidor, comprobaciones de roles, controles de seguridad y registro de auditoría, pero no es un programa completo de operaciones de seguridad. No proporciona un WAF, escaneo de malware, monitoreo de seguridad externo ni una Política de Seguridad de Contenido estricta. Configure un ingreso de confianza y protecciones de alojamiento, revise el modelo de amenazas y pruebe la aplicación desplegada antes del lanzamiento.

Para configuración detallada, propiedad de datos, límites, migraciones y procedimientos operativos, consulte la guía completa de funciones incluida con el proyecto.
