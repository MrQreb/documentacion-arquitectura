```mermaid
stateDiagram-v2
    
    [*] --> Activo
    Activo --> Inactivo : Desactivar
    Inactivo --> Activo : Activar
    Activo --> Eliminado : Eliminar
    Inactivo --> Eliminado : Eliminar
    Eliminado --> [*]
```