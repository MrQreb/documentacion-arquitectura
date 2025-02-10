```mermaid
sequenceDiagram
    participant Usuario
    participant BaseDeDatos

    Usuario->>BaseDeDatos: Registrar Usuario
    BaseDeDatos-->>Usuario: Confirmar registro del Usuario

    Usuario->>BaseDeDatos: Consultar Usuarios
    BaseDeDatos-->>Usuario: Devolver lista de Usuarios

    Usuario->>BaseDeDatos: Actualizar Usuario
    BaseDeDatos-->>Usuario: Confirmar actualización del Usuario

    Usuario->>BaseDeDatos: Eliminar Usuario
    BaseDeDatos-->>Usuario: Confirmar eliminación del Usuario
```    