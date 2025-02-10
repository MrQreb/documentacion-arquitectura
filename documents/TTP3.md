# 1.1 Modularidad

## Backend en Nestjs
El backend esta estrucutado en módulos.

Facilita la gestión, el mantenimiento y la escalabilidad de la aplicación.

### Estructura
***Módulos***: Los módulos son la unidad básica de organización .Cada módulo encapsula un conjunto de funcionalidades relacionadas. Un módulo se define mediante un decorador @Module y puede importar otros módulos, declarar componentes (controladores, servicios, etc.) y exportar componentes para que otros módulos los utilicen.

***Controladores***: Los controladores manejan las solicitudes entrantes y devuelven las respuestas al cliente. Se definen mediante el decorador @Controller.

***Servicios***: Los servicios contienen la lógica de negocio y se pueden inyectar en controladores u otros servicios. Se definen mediante el decorador @Injectable.

***Inyección de Dependencias***: Permite gestionar las dependencias entre los diferentes componentes de la aplicación de manera eficiente.

### Ejemplo de Modulo
```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UsersModule } from './users/users.module';

@Module({
  imports: [UsersModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

# 1.2. Separación de Preocupaciones (Separation of Concerns, SoC)

La Separación de Preocupaciones (Separation of Concerns, SoC) es un principio de diseño que sugiere que una aplicación debe dividirse en distintas secciones, cada una de las cuales aborda una preocupación específica. En NestJS, este principio se aplica de varias maneras:

### Modularidad
NestJS organiza el código en módulos, donde cada módulo encapsula una funcionalidad específica. Esto permite que diferentes partes de la aplicación se desarrollen, mantengan y escalen de manera independiente.

### Controladores
Los controladores (@Controller) se encargan de manejar las solicitudes HTTP y devolver las respuestas adecuadas. Esto separa la lógica de manejo de solicitudes de la lógica de negocio.

### Servicios
Los servicios (@Injectable) contienen la lógica de negocio y se pueden inyectar en controladores u otros servicios. Esto asegura que la lógica de negocio esté separada de la lógica de manejo de solicitudes.

### Inyección de Dependencias
La inyección de dependencias permite gestionar las dependencias entre los diferentes componentes de la aplicación de manera eficiente. Esto facilita la separación de preocupaciones al permitir que los componentes se centren en su funcionalidad específica sin preocuparse por la creación y gestión de sus dependencias.

#### Ejemplo de Separación de Preocupaciones en NestJS

##### Modulo
```typescript
import { Module } from '@nestjs/common';
import { DentistaService } from './dentista.service';
import { DentistaController } from './dentista.controller';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Dentista } from './entities/dentista.entity';
import { AuthModule } from '../auth/auth.module';

@Module({
  controllers: [DentistaController],
  providers: [DentistaService],
  imports:[
    TypeOrmModule.forFeature([Dentista]),
    AuthModule,
  ],
  exports:[DentistaService]
})
export class DentistaModule {}
```
##### Controller
```typescript
import { Controller, Get, Post, Body, Patch, Param, Delete } from '@nestjs/common';
import { DentistaService } from './dentista.service';
import { CreateDentistaDto } from './dto/create-dentista.dto';
import { UpdateDentistaDto } from './dto/update-dentista.dto';
import { Auth } from '../auth/decorators';
import { ValidRoles } from '../auth/interfaces';
import { ApiBearerAuth, ApiOperation, ApiTags } from '@nestjs/swagger';

@ApiTags('Dentista')
@ApiBearerAuth()
@Controller('dentista')
export class DentistaController {
  constructor(private readonly dentistaService: DentistaService) {}

  @ApiOperation({ summary:'Registar nuevos dentistas', description:'Necesita del rol creator'  })
  @Auth(ValidRoles.creator)
  @Post('create')
  create(@Body() createDentistaDto: CreateDentistaDto) {
    return this.dentistaService.create(createDentistaDto)
  }

  @ApiOperation({ summary:'Obtener dentistas con filtros', description:'Necesita del rol creator'  })
  @Auth(ValidRoles.creator)
  @Get('get')
  findAll() {
    return this.dentistaService.findAll();
  }

  @ApiOperation({ summary:'Obtener dentistas por id', description:'Necesita del rol creator'  })
  @Auth(ValidRoles.creator)
  @Get('findOne/:id')
  findOne(@Param('id') id: string) {
    return this.dentistaService.findOne(+id);
  }

  @ApiOperation({ summary:'Actualizar dentista por id', description:'Necesita del rol creator' })
  @Auth(ValidRoles.creator)
  @Patch('update/:id')
  update(@Param('id') id: string, @Body() updateDentistaDto: UpdateDentistaDto) {
    return this.dentistaService.update(+id, updateDentistaDto);
  }

  @ApiOperation({ summary:'Eliinar denista por id',  description:'Necesita del rol creator'  })
  @Auth(ValidRoles.creator)
  @Delete('delete/:id')
  remove(@Param('id') id: string) {
    return this.dentistaService.remove(+id);
  }
}
```

##### Service
```typescript
import { Injectable, NotFoundException, Logger } from '@nestjs/common';
import { CreateDentistaDto } from './dto/create-dentista.dto';
import { UpdateDentistaDto } from './dto/update-dentista.dto';
import { InjectRepository } from '@nestjs/typeorm';
import { Dentista } from './entities/dentista.entity';
import { Repository } from 'typeorm';
import { handleDBExceptions } from '../../common/helpers/handleBDExeptions';
@Injectable()
export class DentistaService {

  private readonly logger = new Logger('dentistaService');

  constructor(
    @InjectRepository(Dentista)
    private readonly dentistaRepository: Repository<Dentista>
  ){}

  async create(createDentistaDto: CreateDentistaDto) {
    
    const dentista = await this.dentistaRepository.create({
      ...createDentistaDto
    });

    await this.dentistaRepository.save(dentista);

    return dentista;
  }

  async findAll() {

    const totalDentistas = await this.dentistaRepository.count();

    const dentistas = await this.dentistaRepository.find({
      take:totalDentistas
    })

    if(!dentistas) throw new NotFoundException(`no se encontraron dentistas`)

    return dentistas;
  }

  async findOne(id: number) {
    
    const dentista = await this.dentistaRepository.findOne({
      where:{dentista_id:id}
    })

    if(!dentista) throw new NotFoundException(`no se encontraro dentista`)

    return dentista;
  }

  async update(id: number, updateDentistaDto: UpdateDentistaDto) {
    
    try{

      const nuevoDentista = await this.dentistaRepository.preload({
        ...updateDentistaDto,
        dentista_id:id
      });

      if (!nuevoDentista) throw new NotFoundException(`Dentista with ID ${id} not found`);
        
      await this.dentistaRepository.save(nuevoDentista);
      return nuevoDentista;
    }catch(error){
      handleDBExceptions(this.logger,error,'dentista')
    }

  }

  async remove(id: number) {
    const dentista:Dentista = await this.findOne(id);

    dentista.dentista_eliminado = true;

    await this.dentistaRepository.save(dentista);

    return { message:'dentista eliminado' }
  }

  async deleteAllDentists(){
    await this.dentistaRepository.clear();
    this.logger.debug('Todos los dentistas eliminados');
    return { message:'Todos los dentistas fueron eliminados' };
  }
}
```

# 1.3. Bajo Acoplamiento y Alta Cohesión

### Bajo Acoplamiento
Inyección de Dependencias: Permite que los componentes (como servicios y controladores) no dependan directamente de las implementaciones de otros componentes, sino de sus interfaces o contratos. Esto facilita el reemplazo y la modificación de componentes sin afectar a otros.

### Módulos
Encapsulan funcionalidades específicas y pueden ser importados y exportados de manera independiente. Esto permite que los módulos sean reutilizables y que los cambios en un módulo no afecten a otros módulos.

## Alta Cohesión

### Servicios
Contienen la lógica de negocio y están diseñados para manejar tareas específicas. Esto asegura que cada servicio tenga una responsabilidad clara y bien definida.

### Controladores
Los controladores (@Controller) manejan las solicitudes HTTP y delegan la lógica de negocio a los servicios. Esto separa claramente la lógica de manejo de solicitudes de la lógica de negocio, asegurando que cada componente tenga una responsabilidad específica.

### Beneficios

**Facilidad de Mantenimiento**: Los cambios en un componente no afectan a otros componentes.

**Reutilización**: Los componentes pueden ser reutilizados en diferentes partes de la aplicación.

**Escalabilidad**: La aplicación puede crecer de manera modular, añadiendo nuevas funcionalidades sin afectar las existentes.

**Pruebas**: Facilita la escritura de pruebas unitarias y de integración, ya que los componentes tienen responsabilidades claras y están desacoplados.

# 1.4. Código Limpio y Buenas Prácticas

Para estar aplicando buenas prácticas y clean code el codigo del backend estará aplicando:

- **Responsabilidad Única**: Cada método en el servicio tiene una - responsabilidad clara y específica.

- **Manejo de Excepciones**: Uso de NotFoundException y handleDBExceptions para manejar errores de manera consistente.

- **Inyección de Dependencias**: Uso de @InjectRepository para inyectar el repositorio de Dentista, lo que facilita la prueba y el mantenimiento del código.

- **Logging**: Uso de Logger para registrar eventos importantes, lo que ayuda en la depuración y monitoreo.

- Nombres de métodos y variales descriptivos

# 1.5. Facilidad de Pruebas (Testabilidad)

Para asegurar la testabilidad del código, estamos utilizando Jest junto con NestJS para realizar pruebas unitarias.

Utilizaremos Jest como la libreria de pruebas. Esto incluye la instalación de las dependencias necesarias y la configuración del entorno de pruebas.

#### Pruebas Unitarias para Servicios
Se crearan archivos de prueba específicos para cada servicio. Utilizamos mocks y stubs para simular dependencias y aislar el código que estamos probando

#### Pruebas Unitarias para Controladores
Las pruebas aseguraran que los controladores manejen las solicitudes HTTP correctamente y deleguen la lógica de negocio a los servicios de manera adecuada.

#### Inyección de Dependencias
Utilizamos la inyección de dependencias para facilitar la prueba y el mantenimiento del código.

# 2.1. Mantenimiento Correctivo

Para poder corregir errores y problemas en la aplicación después de su despliegue.

**Identificación y Registro de Errores**: Utilizamos Logger para registrar eventos importantes y errores, y NotFoundException junto con handleDBExceptions para manejar errores de manera consistente.


**Pruebas y Validación**: Realizamos pruebas unitarias con Jest para asegurar que cada componente funcione correctamente de manera aislada, y pruebas de integración para asegurar que los diferentes módulos y componentes funcionen correctamente juntos.

**Estrategias de Mantenimiento**: Mantenemos las dependencias del proyecto actualizadas y realizamos refactorizaciones periódicas del código para mejorar su legibilidad, mantenibilidad y rendimiento.

El mantenimiento correctivo será eficiente y efectivo, permitiendo que la aplicación se mantenga estable y funcional a lo largo del tiempo.

### 2.2. Mantenimiento Evolutivo

El mantenimiento evolutivo consiste se prodra realizar facilmente gracias a la estuctura del codigo.

Gracias a la estructura modular, podemos agregar nuevas funcionalidades creando nuevos módulos o extendiendo los existentes sin afectar otras partes de la aplicación.

### 2.3. Mantenimiento Preventivo

Con respecto al mantenimiento preventivo se planea:

- **Refactorización**: Realizar refactorizaciones periódicas del código para mejorar su legibilidad, mantenibilidad y rendimiento, asegurando que el código siga siendo limpio y eficiente.
  
- **Documentación**: La documentación será a través de ***Swagger***. Swagger es una herramienta para la documentación y desarrollo de APIs. Proporciona una interfaz interactiva que permite a los desarrolladores explorar y probar las APIs de manera sencilla.
  
- **Pruebas y Validación**: Realizar pruebas unitarias y de integración para asegurar que el código funcione correctamente y detectar posibles problemas antes de que ocurran.

### 2.4. Mantenimiento Adaptativo

Para el mantenimineto adapatavito tendremos en cuenta:

- **Compatibilidad con Nuevas Plataformas**: Ajustar el código para asegurar que sea compatible con nuevas plataformas o sistemas operativos. Al ser una ***API REST*** permite interactuar con cualquier lenguaje de cliente.
  
- **Actualización de Dependencias**: Mantener las dependencias del proyecto actualizadas para asegurar que el software funcione correctamente en el nuevo entorno.
  
- **Pruebas y Validación**: Realizar pruebas en el nuevo entorno para asegurar que el software funcione correctamente y detectar posibles problemas.
  
Lo anterior menciona permite que la aplicación se adapte a los cambios en el negocio, evite futuros problemas y funcione correctamente en nuevos entornos.

# 3.1. Aplicación de Patrones de Diseño

En nestjs nos permite apliar patrones de diseño que permiten organizar mejor el código y facilitar su mantenimiento

**Factory**: NestJS utiliza el patrón Factory para la creación de instancias de servicios y controladores. Esto se logra a través de la inyección de dependencias, donde los proveedores son registrados y gestionados por el contenedor de inyección de dependencias de NestJS.

**Singleton**: Los servicios en NestJS son instancias únicas (singleton) por defecto. Esto significa que una única instancia de un servicio es compartida a lo largo de toda la aplicación, lo que facilita la gestión del estado y la configuración.

**MVC (Model-View-Controller)**: NestJS sigue el patrón MVC, donde los controladores manejan las solicitudes HTTP, los servicios contienen la lógica de negocio y los modelos representan los datos. Esto separa claramente las responsabilidades y facilita el mantenimiento del código.


# 3.2. Uso de Arquitecturas Escalables

NestJS aplica arquitecturas escalables para asegurar que la aplicación pueda crecer y adaptarse a las necesidades cambiantes del negocio

**Arquitectura en Capas**: Organiza el código en capas, separando claramente las responsabilidades. Las capas típicas incluyen la capa de presentación (controladores), la capa de negocio (servicios) y la capa de datos (repositorios). Esto facilita el mantenimiento y la escalabilidad de la aplicación.

# 3.3. Automatización de Pruebas y CI/CD
Para asegurar la calidad y estabilidad del software, implementaremos un pipeline de integración y entrega continua (CI/CD) utilizando Heroku como hosting. Esto nos permitirá detectar fallos de manera temprana y mantener una buena cobertura de pruebas para evitar que los cambios introduzcan errores en el sistema.

## Implementación de CI/CD

- **Pipeline de CI/CD**: Configuraremos un pipeline de CI/CD que automatice el proceso de construcción, prueba y despliegue de la aplicación. Esto incluye la integración con herramientas de control de versiones como GitHub y la configuración de workflows en Heroku.

- **Automatización de Pruebas**: Utilizaremos Jest junto con NestJS para realizar pruebas unitarias y de integración de manera automática en cada commit. Esto asegurará que cada cambio en el código sea probado exhaustivamente antes de ser desplegado.

- **Despliegue Automático**: Configuraremos Heroku para que despliegue automáticamente la aplicación después de que las pruebas hayan pasado exitosamente. Esto asegurará que la versión más reciente y estable de la aplicación esté siempre en producción.

# 3.4. Documentación Clara y Precisa
Para asegurar una documentación clara y precisa, estamos utilizando Swagger para documentar nuestras APIs de manera automática. Swagger es una herramienta que genera documentación interactiva a partir del código fuente, asegurando que la documentación esté siempre actualizada. Facilita a los desarrolladores explorar y probar las APIs.

- **Automatización**: Swagger genera automáticamente la documentación de la API a partir del código fuente, lo que reduce el esfuerzo manual y asegura que la documentación esté siemprea actualizada.
  
- **Interfaz Interactiva**: Proporciona una interfaz web interactiva donde los desarrolladores pueden explorar y probar las diferentes rutas y métodos de la API, mejorando la experencia de desarrollo.

- **Generación de Código**: Swagger permite la generación de código cliente y servidor en varios lenguajes de programación a partir de la especificación de la API, lo que reduce errores y acelera el desarrollo.