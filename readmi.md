# Taller Mecánico - Modelo Orientado a Objetos Seguro

**Módulo:** Programación Orientada a Objetos Segura  
**Periodo:** Segundo Semestre, 2026  
**Institución / Contexto:** Ejercicio académico de modelado de software

---

## 1. Descripción del Proyecto

Este repositorio contiene el diseño, especificación e implementación del **modelo completo de un Taller Mecánico**. El proyecto tiene como propósito aplicar principios avanzados de la **Programación Orientada a Objetos (POO)** incorporando directrices de **desarrollo seguro** (validación rigurosa de entradas, encapsulamiento robusto, control de estados y consistencia de datos).

---

## 2. Objetivos Generales

- Modelar las entidades centrales involucradas en la operación de un taller mecánico (clientes, vehículos, personal técnico, órdenes de trabajo, repuestos y facturación).
- Garantizar la integridad de los datos mediante encapsulamiento y validación en las capas del modelo.
- Implementar relaciones adecuadas de herencia, agregación y composición según las reglas de negocio del taller.
- Aplicar estándares de seguridad en POO para evitar inconsistencias en el ciclo de vida de los servicios y reparaciones.

---

## 3. Modelo de Dominio y Entidades Principales

### 3.1. Personas y Usuarios
- **`Persona` (Clase Abstracta):** Base común con atributos protegidos y validados (`rut`, `nombreCompleto`, `telefono`, `email`).
- **`Cliente`:** Especialización de `Persona`. Mantiene la lista de vehículos asociados y el historial de atención en el taller.
- **`Empleado`:** Especialización de `Persona` con datos laborales (`idEmpleado`, `cargo`, `fechaContratacion`).
- **`Mecanico`:** Especialización de `Empleado` responsable de las reparaciones (`especialidad`, `certificaciones`, `disponible`).

### 3.2. Vehículos
- **`Vehiculo`:** Representa los automóviles ingresados al taller.
  - Atributos: `patente` (identificador único normalizado), `marca`, `modelo`, `anio`, `kilometraje`, `duenio` (asociación con `Cliente`).

### 3.3. Gestión del Trabajo y Reparaciones
- **`OrdenDeTrabajo`:** Entidad central que coordina el servicio.
  - Atributos: `idOrden`, `fechaIngreso`, `fechaEstimadaEntrega`, `estado` (`PENDIENTE`, `EN_DIAGNOSTICO`, `EN_REPARACION`, `FINALIZADO`, `ENTREGADO`), `vehiculo`, `mecanicoAsignado`, `serviciosRealizados`, `repuestosUtilizados`, `costoTotal`.
- **`Servicio`:** Tareas o prestaciones realizadas (`idServicio`, `descripcion`, `tiempoEstimado`, `precioManoObra`).
- **`Repuesto`:** Piezas o insumos utilizados (`codigoPieza`, `descripcion`, `precioUnitario`, `stockDisponible`).

### 3.4. Facturación y Cobranza
- **`Factura`:** Comprobante de pago generado al completar la orden de trabajo (`folio`, `fechaEmision`, `subtotal`, `iva`, `total`, `metodoPago`, `ordenAsociada`).

---

## 4. Diagrama de Clases (Mermaid)

```mermaid
classDiagram
    direction TB

    class Persona {
        <<abstract>>
        -string rut
        -string nombreCompleto
        -string telefono
        -string email
        +getRut() string
        +setTelefono(string) void
        +setEmail(string) void
    }

    class Cliente {
        -List~Vehiculo~ vehiculos
        +registrarVehiculo(Vehiculo) void
        +getHistorialOrdenes() List
    }

    class Empleado {
        -string idEmpleado
        -string cargo
        +getIdEmpleado() string
    }

    class Mecanico {
        -string especialidad
        -bool disponible
        +asignarOrden(OrdenDeTrabajo) void
        +liberar() void
    }

    class Vehiculo {
        -string patente
        -string marca
        -string modelo
        -int anio
        -int kilometraje
        -Cliente propietario
        +actualizarKilometraje(int) void
    }

    class OrdenDeTrabajo {
        -string idOrden
        -DateTime fechaIngreso
        -DateTime fechaEntrega
        -EstadoOrden estado
        -Vehiculo vehiculo
        -Mecanico mecanico
        -List~Servicio~ servicios
        -List~Repuesto~ repuestos
        +agregarServicio(Servicio) void
        +agregarRepuesto(Repuesto, int) void
        +cambiarEstado(EstadoOrden) void
        +calcularTotal() decimal
    }

    class Servicio {
        -string idServicio
        -string nombre
        -decimal costoManoObra
        +getCosto() decimal
    }

    class Repuesto {
        -string codigo
        -string nombre
        -decimal precioUnitario
        -int stock
        +descontarStock(int) bool
    }

    class Factura {
        -string folio
        -DateTime fechaEmision
        -decimal subtotal
        -decimal iva
        -decimal total
        -string metodoPago
        +generarDetalle() string
    }

    Persona <|-- Cliente
    Persona <|-- Empleado
    Empleado <|-- Mecanico

    Cliente "1" o-- "0..*" Vehiculo : es propietario de
    Vehiculo "1" <-- "1..*" OrdenDeTrabajo : atendido en
    Mecanico "1" <-- "0..*" OrdenDeTrabajo : asignado a
    OrdenDeTrabajo "1" *-- "1..*" Servicio : compone
    OrdenDeTrabajo "1" *-- "0..*" Repuesto : utiliza
    OrdenDeTrabajo "1" -- "1" Factura : genera
```

---

## 5. Principios de POO Segura Aplicados

1. **Encapsulamiento estricto:** Todos los atributos críticos son privados (`private`) y se accede a ellos mediante métodos accesores controlados con validaciones (e.g., formato de RUT, patentes válidas, montos positivos).
2. **Control de estados inmutable en la Orden de Trabajo:** Las transiciones de estado (`PENDIENTE` $\rightarrow$ `EN_DIAGNOSTICO` $\rightarrow$ `EN_REPARACION` $\rightarrow$ `FINALIZADO`) impiden saltos inválidos que puedan generar cobros indebidos o entregas no autorizadas.
3. **Manejo seguro del stock:** Descuento atómico de piezas y repuestos para prevenir inconsistencias de inventario.
4. **Separación de responsabilidades (SRP):** Desacoplamiento entre la lógica de ejecución del servicio técnico y la generación tributaria/financiera en la factura.
