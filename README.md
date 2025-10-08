# nest-practice

-----

### **1. Configuración inicial**

Antes de empezar, necesitas tener Node.js y npm (o yarn) instalados. También, NestJS tiene su propia interfaz de línea de comandos (CLI) que simplifica la creación de proyectos y la generación de componentes.

1.  **Instalar Node.js y npm:** Si no los tienes, descárgalos desde el sitio web oficial de Node.js.
2.  **Instalar la CLI de NestJS:** Abre tu terminal y ejecuta el siguiente comando globalmente para poder usarlo en cualquier lugar:

<!-- end list -->

```bash
npm i -g @nestjs/cli
```

-----

### **2. Creación del proyecto**

Una vez que la CLI de NestJS esté instalada, puedes crear un nuevo proyecto con el comando `nest new`.

1.  **Crear un nuevo proyecto:** Navega a la carpeta donde quieres crear tu proyecto y ejecuta:

<!-- end list -->

```bash
nest new project-name
```

Reemplaza `project-name` con el nombre que prefieras para tu proyecto. La CLI te preguntará qué gestor de paquetes usar (`npm`, `yarn`, `pnpm`). Selecciona `npm`. Esto creará una estructura de carpetas básica con un proyecto funcional de "hola mundo".

2.  **Iniciar el servidor de desarrollo:** Una vez que la instalación esté completa, navega a la carpeta del proyecto y ejecuta:

<!-- end list -->

```bash
cd project-name
npm run start:dev
```

Esto iniciará el servidor de desarrollo. Puedes abrir `http://localhost:3000` en tu navegador para ver la respuesta "Hello World\!".

-----

### **3. Configuración de la base de datos (PostgreSQL)**

Para este proyecto, usaremos **PostgreSQL** y **TypeORM** como ORM (Object-Relational Mapper).

1.  **Instalar dependencias de la base de datos:** Instala el paquete de PostgreSQL y TypeORM en tu proyecto.

<!-- end list -->

```bash
npm install @nestjs/typeorm typeorm pg
```

2.  **Configurar el módulo raíz de la base de datos:** En el archivo `src/app.module.ts`, importa y configura `TypeOrmModule`. **Este es un ejemplo de una configuración simple para una base de datos local.**

<!-- end list -->

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'tu_usuario',
      password: 'tu_contraseña',
      database: 'tu_base_de_datos',
      entities: [__dirname + '/**/*.entity{.ts,.js}'],
      synchronize: true, // ¡No usar en producción! Esto sincroniza el esquema de la base de datos automáticamente.
    }),
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Asegúrate de reemplazar `tu_usuario`, `tu_contraseña` y `tu_base_de_datos` con tus credenciales de PostgreSQL.

-----

### **4. Creación de módulos y entidades**

Para demostrar una buena complejidad, crearemos dos módulos: **`Usuarios`** y **`Productos`**.

1.  **Generar el módulo de Usuarios:**

<!-- end list -->

```bash
nest generate module usuarios
```

Esto creará la carpeta `src/usuarios`.

2.  **Generar la entidad de Usuario:** En la carpeta `src/usuarios`, crea un archivo `usuario.entity.ts`. Esto define la estructura de la tabla en la base de datos.

<!-- end list -->

```typescript
// src/usuarios/usuario.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Usuario {
  @PrimaryGeneratedColumn()
  id: number;

  @Column({ unique: true })
  nombreUsuario: string;

  @Column()
  email: string;

  @Column()
  fechaRegistro: Date;
}
```

3.  **Generar el módulo de Productos:**

<!-- end list -->

```bash
nest generate module productos
```

Esto creará la carpeta `src/productos`.

4.  **Generar la entidad de Producto:** En la carpeta `src/productos`, crea un archivo `producto.entity.ts`.

<!-- end list -->

```typescript
// src/productos/producto.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity()
export class Producto {
  @PrimaryGeneratedColumn()
  id: number;

  @Column()
  nombre: string;

  @Column('float')
  precio: number;

  @Column()
  stock: number;
}
```

-----

### **5. Conexión de entidades y módulos**

Ahora necesitamos conectar las entidades a sus respectivos módulos para que NestJS y TypeORM las reconozcan.

1.  **Configurar el módulo de Usuarios:** En `src/usuarios/usuarios.module.ts`, importa `TypeOrmModule` y pasa la entidad `Usuario`.

<!-- end list -->

```typescript
// src/usuarios/usuarios.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Usuario } from './usuario.entity';
import { UsuariosController } from './usuarios.controller';
import { UsuariosService } from './usuarios.service';

@Module({
  imports: [TypeOrmModule.forFeature([Usuario])],
  controllers: [UsuariosController],
  providers: [UsuariosService],
})
export class UsuariosModule {}
```

2.  **Configurar el módulo de Productos:** Haz lo mismo para el módulo de `Productos`.

<!-- end list -->

```typescript
// src/productos/productos.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { Producto } from './producto.entity';
import { ProductosController } from './productos.controller';
import { ProductosService } from './productos.service';

@Module({
  imports: [TypeOrmModule.forFeature([Producto])],
  controllers: [ProductosController],
  providers: [ProductosService],
})
export class ProductosModule {}
```

3.  **Importar los nuevos módulos en el módulo raíz:** En `src/app.module.ts`, importa los módulos `UsuariosModule` y `ProductosModule` para que la aplicación los reconozca.

<!-- end list -->

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { UsuariosModule } from './usuarios/usuarios.module';
import { ProductosModule } from './productos/productos.module';
import { Usuario } from './usuarios/usuario.entity';
import { Producto } from './productos/producto.entity';

@Module({
  imports: [
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'tu_usuario',
      password: 'tu_contraseña',
      database: 'tu_base_de_datos',
      entities: [Usuario, Producto], // Asegúrate de que las entidades estén aquí
      synchronize: true,
    }),
    UsuariosModule,
    ProductosModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

-----

### **6. Creación de servicios y controladores**

Ahora crearemos la lógica de negocio para manejar las operaciones **CRUD** (Crear, Leer, Actualizar, Borrar).

1.  **Generar servicios y controladores para Usuarios:**

<!-- end list -->

```bash
nest generate service usuarios
nest generate controller usuarios
```

Esto creará `usuarios.service.ts` y `usuarios.controller.ts` dentro de la carpeta `src/usuarios`.

2.  **Codificar el servicio de Usuarios:** El servicio interactúa directamente con la base de datos.

<!-- end list -->

```typescript
// src/usuarios/usuarios.service.ts
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Usuario } from './usuario.entity';

@Injectable()
export class UsuariosService {
  constructor(
    @InjectRepository(Usuario)
    private usuariosRepository: Repository<Usuario>,
  ) {}

  findAll(): Promise<Usuario[]> {
    return this.usuariosRepository.find();
  }

  findOne(id: number): Promise<Usuario> {
    return this.usuariosRepository.findOneBy({ id });
  }

  async create(usuario: Usuario): Promise<Usuario> {
    return this.usuariosRepository.save(usuario);
  }

  async update(id: number, usuarioActualizado: Partial<Usuario>): Promise<Usuario> {
    await this.usuariosRepository.update(id, usuarioActualizado);
    return this.findOne(id);
  }

  async remove(id: number): Promise<void> {
    await this.usuariosRepository.delete(id);
  }
}
```

3.  **Codificar el controlador de Usuarios:** El controlador maneja las peticiones HTTP y usa el servicio para la lógica de negocio.

<!-- end list -->

```typescript
// src/usuarios/usuarios.controller.ts
import { Controller, Get, Post, Body, Param, Put, Delete } from '@nestjs/common';
import { UsuariosService } from './usuarios.service';
import { Usuario } from './usuario.entity';

@Controller('usuarios')
export class UsuariosController {
  constructor(private readonly usuariosService: UsuariosService) {}

  @Get()
  findAll(): Promise<Usuario[]> {
    return this.usuariosService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string): Promise<Usuario> {
    return this.usuariosService.findOne(+id);
  }

  @Post()
  create(@Body() usuario: Usuario): Promise<Usuario> {
    return this.usuariosService.create(usuario);
  }

  @Put(':id')
  update(@Param('id') id: string, @Body() usuarioActualizado: Usuario): Promise<Usuario> {
    return this.usuariosService.update(+id, usuarioActualizado);
  }

  @Delete(':id')
  remove(@Param('id') id: string): Promise<void> {
    return this.usuariosService.remove(+id);
  }
}
```

4.  **Repetir el proceso para Productos:** Sigue los mismos pasos para crear el servicio y controlador de `Productos` con su lógica CRUD.

-----

### **7. Ejecución y prueba**

1.  **Asegúrate de que la base de datos esté corriendo:** Inicia tu servidor de PostgreSQL.
2.  **Inicia el servidor de NestJS:**

<!-- end list -->

```bash
npm run start:dev
```

3.  **Prueba la API:** Puedes usar herramientas como **Postman** o **Insomnia** para enviar peticiones a los siguientes endpoints (puntos finales):

<!-- end list -->

  * **Usuarios:**
      * `GET http://localhost:3000/usuarios` (obtener todos los usuarios)
      * `GET http://localhost:3000/usuarios/1` (obtener un usuario por ID)
      * `POST http://localhost:3000/usuarios` (crear un nuevo usuario, enviar JSON en el cuerpo)
      * `PUT http://localhost:3000/usuarios/1` (actualizar un usuario)
      * `DELETE http://localhost:3000/usuarios/1` (borrar un usuario)
  * **Productos:**
      * `GET http://localhost:3000/productos`
      * `POST http://localhost:3000/productos`
