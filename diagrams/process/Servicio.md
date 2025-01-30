```mermaid
sequenceDiagram
    participant Dentista
    participant BaseDeDatos

    Dentista->>BaseDeDatos: Registrar Servicio (asociado al Paciente)
    BaseDeDatos-->>Dentista: Confirmar registro del Servicio

    Dentista->>BaseDeDatos: Consultar Servicios
    BaseDeDatos-->>Dentista: Devolver lista de Servicios

    Dentista->>BaseDeDatos: Actualizar Servicio
    BaseDeDatos-->>Dentista: Confirmar actualización del Servicio

    Dentista->>BaseDeDatos: Eliminar Servicio
    BaseDeDatos-->>Dentista: Confirmar eliminación del Servicio
```