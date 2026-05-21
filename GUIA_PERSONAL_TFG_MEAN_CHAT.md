# Guia Personal del TFG - Mean Chat

## 1. Resumen ejecutivo
Mean Chat es una aplicacion web full stack de mensajeria en tiempo real.
Su objetivo es demostrar dominio tecnico de frontend, backend, base de datos, seguridad y despliegue reproducible.

## 2. Objetivo del proyecto
- Construir un sistema de chat realista, util y defendible en un tribunal.
- Implementar autenticacion segura con control de acceso.
- Gestionar chats privados y grupales con historial paginado.
- Resolver sincronizacion en tiempo real entre multiples clientes.
- Facilitar despliegue y evaluacion con Docker Compose.

## 3. Stack tecnologico
- Frontend: Angular 21 + Ionic + TypeScript + Socket.IO client.
- Backend: Node.js + Express + Socket.IO + Mongoose + JWT.
- Base de datos: MongoDB.
- Infraestructura local: Docker Compose con 3 servicios.
- Seguridad: rate limiting, middleware de autenticacion, verificacion de membresia, CORS por entorno.

## 4. Arquitectura general
El sistema se divide en tres capas:
- Capa de presentacion: cliente Angular/Ionic.
- Capa de negocio: API y sockets en Node.js/Express.
- Capa de datos: MongoDB con modelos Mongoose.

Canales de comunicacion:
- REST para operaciones tradicionales como login o consulta de recursos.
- Socket.IO para eventos en tiempo real, como envio y recepcion instantanea de mensajes.

## 5. Modelo de datos
Entidades principales:
- User: id, username, passwordHash, name, surname, email.
- Chat: id, isGroup, members, name, lastMessage, createdAt, updatedAt.
- Message: id, chat, sender, content, type, createdAt.

Relaciones:
- Un usuario puede pertenecer a muchos chats.
- Un chat puede tener muchos usuarios.
- Un chat puede contener muchos mensajes.
- Un usuario puede enviar muchos mensajes.

## 6. API REST implementada
Autenticacion:
- POST /auth/register
- POST /auth/login
- POST /auth/refresh
- POST /auth/logout
- GET /auth/me
- GET /auth/users

Chat:
- POST /chat
- GET /chat
- GET /chat/:id
- GET /chat/:id/messages?page=1&limit=30

Nota:
- El envio de mensajes en tiempo real se realiza por sockets, no por endpoint REST.

## 7. Eventos Socket.IO
Eventos cliente a servidor:
- join-chat
- leave-chat
- chat-message

Eventos servidor a cliente:
- chat-created
- chat-updated
- chat-message
- user-joined
- user-left

Valor de estos eventos:
- Sincronizacion inmediata de mensajes.
- Actualizacion en vivo de la lista de chats.
- Mejor experiencia de usuario sin recargas.

## 8. Flujo de autenticacion
- El usuario inicia sesion con credenciales.
- El backend valida contrasena con hash y devuelve access token + refresh token.
- El frontend adjunta el access token en peticiones protegidas.
- Si una peticion responde 401, el frontend intenta renovar sesion con /auth/refresh.
- Si refresh es valido, guarda tokens nuevos y reintenta la peticion original.
- Si refresh falla, limpia tokens y redirige a login.
- Middleware de autenticacion valida el token.
- Si es valido, permite acceso. Si no, devuelve error.

## 9. Flujo de mensajeria en tiempo real
- El cliente emite chat-message.
- El servidor valida que el usuario pertenezca al chat.
- El mensaje se guarda en MongoDB.
- El servidor emite mensaje a la sala.
- Todos los clientes conectados reciben y actualizan interfaz al instante.
- Ademas se emite chat-updated para mantener orden y ultimo mensaje en listado.

## 10. Seguridad aplicada
- JWT con expiracion para sesiones.
- Rotacion de refresh tokens y revocacion en logout.
- Middleware centralizado de autorizacion.
- Verificacion de membresia antes de unirse o escribir en chat.
- Rate limiting en rutas de autenticacion.
- CORS configurable por entorno.
- Swagger solo habilitado fuera de produccion.

## 11. Docker y despliegue local
Servicios en Compose:
- mongo
- backend
- frontend

Caracteristicas:
- Arranque coordinado del entorno completo.
- Variables de entorno para configuracion.
- MongoDB fijado en version 4.4 por compatibilidad.
- Entorno reproducible para evaluacion y demo.

## 12. Validacion y pruebas
Validacion funcional manual:
- Registro e inicio de sesion.
- Creacion de chat.
- Envio y recepcion en tiempo real.
- Consulta de historial paginado.
- Pruebas de acceso no autorizado.

Estado de pruebas automaticas:
- Estructura de testing disponible.
- Cobertura actual aun limitada.
- Mejora prioritaria para evolucion del proyecto.

## 13. Retos tecnicos y soluciones
- Lista de chats sin refresco inmediato.
  - Solucion: eventos chat-created y chat-updated.
- Riesgo de acceso indebido por socket.
  - Solucion: validacion estricta de membresia.
- Problemas en contenedor por dependencia nativa.
  - Solucion: uso de bcryptjs.
- Compatibilidad de base de datos en entorno contenedorizado.
  - Solucion: version de MongoDB fijada a 4.4.

## 14. Limitaciones actuales
- Sin videollamadas ni notificaciones push.
- Sin cifrado extremo a extremo.
- Cobertura de tests automaticos mejorable.
- Metricas formales de rendimiento aun pendientes.

## 15. Roadmap propuesto
Prioridad alta:
- Ampliar tests de backend y frontend.
- Configurar pipeline CI/CD.

Prioridad media:
- Presencia en linea y ultima conexion.
- Notificaciones push.
- Endurecimiento para produccion.

Prioridad exploratoria:
- Cifrado E2E.
- Cliente movil nativo.

## 16. Conclusion personal
Mean Chat demuestra capacidad de disenar, implementar y desplegar una solucion full stack real con requisitos de tiempo real y seguridad.
El proyecto es funcional, mantenible y tiene una base solida para evolucionar a un producto de mayor madurez.