```mermaid
stateDiagram-v2
    [*] --> RegistrarServicio
    RegistrarServicio --> ConfirmarRegistro
    ConfirmarRegistro --> [*]

    [*] --> ConsultarServicios
    ConsultarServicios --> VerificarExistenciaConsulta
    VerificarExistenciaConsulta --> DevolverListaServicios : Servicio encontrado
    VerificarExistenciaConsulta --> NotificarNoEncontradoConsulta : Servicio no encontrado
    DevolverListaServicios --> [*]
    NotificarNoEncontradoConsulta --> [*]

    [*] --> ActualizarServicio
    ActualizarServicio --> VerificarExistenciaActualizacion
    VerificarExistenciaActualizacion --> ConfirmarActualizacion : Servicio encontrado
    VerificarExistenciaActualizacion --> NotificarNoEncontradoActualizacion : Servicio no encontrado
    ConfirmarActualizacion --> [*]
    NotificarNoEncontradoActualizacion --> [*]

    [*] --> EliminarServicio
    EliminarServicio --> VerificarExistenciaEliminacion
    VerificarExistenciaEliminacion --> ConfirmarEliminacion : Servicio encontrado
    VerificarExistenciaEliminacion --> NotificarNoEncontradoEliminacion : Servicio no encontrado
    ConfirmarEliminacion --> [*]
    NotificarNoEncontradoEliminacion --> [*]
```