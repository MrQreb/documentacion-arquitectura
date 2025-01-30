```mermaid
sequenceDiagram
    participant Dentista
    participant BaseDeDatos

    Dentista->>BaseDeDatos: Crear Dentista
    BaseDeDatos-->>Dentista: Confirmar creación del Dentista

    Dentista->>BaseDeDatos: Consultar Dentistas
    BaseDeDatos-->>Dentista: Devolver lista de Dentistas

    Dentista->>BaseDeDatos: Actualizar Dentista
    BaseDeDatos-->>Dentista: Confirmar actualización del Dentista

    Dentista->>BaseDeDatos: Eliminar Dentista
    BaseDeDatos-->>Dentista: Confirmar eliminación del Dentista
```