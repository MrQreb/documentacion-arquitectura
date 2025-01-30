```mermaid
sequenceDiagram
    participant Dentista
    participant BaseDeDatos

    Dentista->>BaseDeDatos: Crear Local (asociado al Dentista)
    BaseDeDatos-->>Dentista: Confirmar creación del Local

    Dentista->>BaseDeDatos: Consultar Locales
    BaseDeDatos-->>Dentista: Devolver lista de Locales

    Dentista->>BaseDeDatos: Actualizar Local
    BaseDeDatos-->>Dentista: Confirmar actualización del Local

    Dentista->>BaseDeDatos: Eliminar Local
    BaseDeDatos-->>Dentista: Confirmar eliminación del Local
```