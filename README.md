# Proyecto-Modulo-2-AC
## Descripción
Sistema de gestión de reservas desarrollado con arquitectura de microservicios en la nube (AWS), diseñado para permitir a los usuarios reservar espacios de manera eficiente, segura y escalable. El sistema maneja desde la autenticación de usuarios hasta el procesamiento de pagos y notificaciones automáticas.
### Problema que Resuelve
Las empresas necesitan gestionar eficientemente sus espacios compartidos (salas de reuniones, espacios de cowork, oficinas privadas) permitiendo a empleados y visitantes reservar recursos de forma simple y evitando conflictos de horarios.
### Solución
Plataforma cloud-native que ofrece:

- Reservas en tiempo real con validación de disponibilidad
- Sistema de pagos integrado
- Notificaciones automáticas
- Escalabilidad automática según demanda
- Alta disponibilidad (99.95% uptime)
- Seguridad enterprise-grade

## Características
### Funcionalidades Core
#### Autenticación y Autorización
Registro y login de usuarios
JWT tokens con refresh automático
Control de acceso basado en roles (RBAC)
Autenticación multifactor (MFA) opcional

#### Gestión de Espacios
CRUD completo de espacios
Categorización (sala de reuniones, cowork, oficina privada)
Búsqueda y filtrado avanzado
Gestión de recursos (proyector, pizarra, etc.)

#### Sistema de Reservas
Creación de reservas con validación en tiempo real
Prevención de conflictos de horario
Modificación y cancelación con políticas configurables
Historial completo de reservas
Cálculo automático de precios

#### Procesamiento de Pagos
Integración con Stripe
Múltiples métodos de pago
Procesamiento de reembolsos
Registro completo de transacciones

#### Sistema de Notificaciones
Confirmación inmediata de reservas
Recordatorios automáticos (24h antes)
Notificaciones de cambios
Soporte para email y SMS

### Características No Funcionales

- Performance: Tiempo de respuesta < 200ms (P95)
- Escalabilidad: Soporta 10,000+ usuarios concurrentes
- Seguridad: Encriptación end-to-end, cumplimiento GDPR
- Disponibilidad: 99.95% uptime con multi-AZ deployment
- Observabilidad: Logging centralizado, tracing distribuido
- CI/CD: Despliegues múltiples diarios sin downtime

## Diagrama de Clases 
El siguiente diagrama muestra la estructura de clases del sistema, incluyendo las entidades principales, servicios, repositorios y controladores:

```mermaid
classDiagram
    %% Capa de Dominio
    class Usuario {
        -String id
        -String nombre
        -String email
        -String password
        -TipoUsuario tipo
        +autenticar()
        +actualizarPerfil()
    }

    class Espacio {
        -String id
        -String nombre
        -TipoEspacio tipo
        -int capacidad
        -List~Recurso~ recursos
        -boolean disponible
        +verificarDisponibilidad(fechaInicio, fechaFin)
        +actualizarEstado()
    }

    class Reserva {
        -String id
        -String usuarioId
        -String espacioId
        -DateTime fechaInicio
        -DateTime fechaFin
        -EstadoReserva estado
        -double precio
        +crear()
        +cancelar()
        +modificar()
        +validarDisponibilidad()
    }

    class DisponibilidadService {
        -ReservaRepository reservaRepo
        +consultarDisponibilidad(espacioId, fecha)
        +bloquearEspacio(reserva)
        +liberarEspacio(reservaId)
    }

    class ReservaService {
        -ReservaRepository reservaRepo
        -DisponibilidadService disponibilidadService
        -NotificacionService notificacionService
        +crearReserva(datos)
        +consultarReserva(id)
        +cancelarReserva(id)
        +listarReservasUsuario(usuarioId)
    }

    class AutenticacionService {
        -UsuarioRepository usuarioRepo
        -TokenService tokenService
        +login(email, password)
        +logout(token)
        +verificarToken(token)
        +renovarToken(token)
    }

    class NotificacionService {
        +enviarConfirmacion(reserva)
        +enviarRecordatorio(reserva)
        +enviarCancelacion(reserva)
    }

    class PagoService {
        -PasarelaPago pasarela
        +procesarPago(reserva)
        +reembolsar(reserva)
        +verificarPago(transaccionId)
    }

    %% Repositorios
    class ReservaRepository {
        <<interface>>
        +guardar(reserva)
        +buscarPorId(id)
        +buscarPorUsuario(usuarioId)
        +buscarPorEspacio(espacioId, fecha)
        +eliminar(id)
    }

    class EspacioRepository {
        <<interface>>
        +guardar(espacio)
        +buscarPorId(id)
        +listarDisponibles()
        +buscarPorTipo(tipo)
    }

    class UsuarioRepository {
        <<interface>>
        +guardar(usuario)
        +buscarPorId(id)
        +buscarPorEmail(email)
    }

    %% Controladores API
    class ReservaController {
        -ReservaService reservaService
        +POST crearReserva(request)
        +GET obtenerReserva(id)
        +DELETE cancelarReserva(id)
        +GET listarReservas(usuarioId)
    }

    class EspacioController {
        -EspacioService espacioService
        +GET listarEspacios()
        +GET consultarDisponibilidad(espacioId, fecha)
        +GET obtenerDetalles(id)
    }

    class UsuarioController {
        -UsuarioService usuarioService
        +POST registrar(datos)
        +GET obtenerPerfil(id)
        +PUT actualizarPerfil(id, datos)
    }

    %% Enumeraciones
    class EstadoReserva {
        <<enumeration>>
        PENDIENTE
        CONFIRMADA
        CANCELADA
        COMPLETADA
    }

    class TipoEspacio {
        <<enumeration>>
        SALA_REUNIONES
        COWORK
        OFICINA_PRIVADA
        SALA_CONFERENCIAS
    }

    class TipoUsuario {
        <<enumeration>>
        CLIENTE
        ADMINISTRADOR
        OPERADOR
    }

    %% Relaciones
    Usuario "1" -- "*" Reserva : realiza
    Espacio "1" -- "*" Reserva : tiene
    Reserva --> EstadoReserva
    Espacio --> TipoEspacio
    Usuario --> TipoUsuario

    ReservaController --> ReservaService
    EspacioController --> DisponibilidadService
    UsuarioController --> AutenticacionService

    ReservaService --> ReservaRepository
    ReservaService --> DisponibilidadService
    ReservaService --> NotificacionService
    ReservaService --> PagoService

    DisponibilidadService --> ReservaRepository
    DisponibilidadService --> EspacioRepository

    AutenticacionService --> UsuarioRepository
```

### Descripción de Componentes

#### Entidades de Dominio
Usuario: Representa un usuario del sistema (cliente, administrador u operador)
Espacio: Define un espacio reservable (sala, cowork, etc.)
Reserva: Registra una reservación de espacio por un usuario

#### Servicios
AutenticacionService: Maneja login, logout y validación de tokens
ReservaService: Lógica de negocio para reservas
DisponibilidadService: Valida y gestiona disponibilidad de espacios
PagoService: Procesa transacciones de pago
NotificacionService: Envía notificaciones a usuarios

#### Repositorios
Interfaces que abstraen el acceso a datos
Implementaciones concretas para PostgreSQL

#### Controladores
Endpoints REST que exponen la funcionalidad
Validación de entrada y manejo de respuestas
