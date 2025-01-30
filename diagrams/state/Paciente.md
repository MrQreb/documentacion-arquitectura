```mermaid
stateDiagram-v2
    [*] --> Disponible
    Disponible --> Reservada : Reservar
    Reservada --> Disponible : Cancelar
    Disponible --> Eliminado : Eliminar
    Reservada --> Eliminado : Eliminar
    Eliminado --> [*]
```