# Persistencia
# LABORATORIO 7: DATA - PERSISTENCIA CON JPA Y MICROSERVICIO DE IMÁGENES CON MONGODB

**Escuela Colombiana de Ingeniería Julio Garavito**  
**Curso:** Desarrollo y Operaciones de Software - DOSW  
**Caso de estudio:** **TechCup**

En este laboratorio, cada equipo (SQUAD) deberá extender su proyecto construido en Java con Spring Boot y Maven para incorporar persistencia de datos en dos frentes:

- **Persistencia relacional** del modelo principal del sistema usando **JPA** y una base de datos relacional, preferiblemente **PostgreSQL**.
- **Persistencia NoSQL** mediante un segundo proyecto que implemente un **microservicio de imágenes** usando **MongoDB**.

Además, el sistema principal deberá tener configurada una base de datos **H2 en memoria** para la ejecución de pruebas automatizadas.

---

## 1. Tabla de contenido

1. [Objetivo](#1-tabla-de-contenido)
2. [Resultados de aprendizaje](#2-resultados-de-aprendizaje)
3. [Estructura general de la solución](#3-estructura-general-de-la-solución)
4. [Parte A – Persistencia relacional con JPA](#parte-a--persistencia-relacional-con-jpa)
5. [Parte B – Microservicio de imágenes con MongoDB](#parte-b--microservicio-de-imágenes-con-mongodb)
6. [Requisitos mínimos del laboratorio](#6-requisitos-mínimos-del-laboratorio)
7. [Evidencias a entregar](#7-evidencias-a-entregar)

---

## 2. Resultados de aprendizaje

Al finalizar el laboratorio, el estudiante estará en capacidad de:

- Configurar dependencias de persistencia en un proyecto Spring Boot con Maven.
- Mapear entidades del dominio usando JPA.
- Persistir información en PostgreSQL.
- Ejecutar pruebas de repositorio e integración con H2.
- Construir un microservicio independiente en Spring Boot para almacenar imágenes en MongoDB.
- Contrastar el uso de bases de datos relacionales y NoSQL dentro de una arquitectura distribuida.

---

## 3. Estructura general de la solución

La entrega estará compuesta por **dos repositorios independientes**.

### Repositorio 1: Proyecto principal (Cree uno con la misma estructura del Laboratorio 6 o puede usarlo para desarrollar esta laboratorio)

Debe contener:

- Aplicación principal en Spring Boot.
- Entidades JPA.
- Repositorios Spring JPA.
- Conexión a PostgreSQL.
- Configuración H2 para pruebas.
- Pruebas automatizadas.

### Repositorio 2: Microservicio de imágenes (uno nuevo con la misma estructura del Laboratorio 6)

Debe contener:

- Proyecto Spring Boot independiente.
- Conexión a MongoDB.
- Documento para almacenar imágenes.
- Repositorio Mongo.
- Controlador REST para cargar, consultar, listar y eliminar imágenes.

---

# PARTE A – PERSISTENCIA RELACIONAL CON JPA

> **Convención de trabajo:** cada paso se desarrolla en su propia rama y se integra a `develop` mediante un Pull Request (PR). **Todo PR debe ser revisado y aprobado por un integrante del equipo diferente a quien lo generó.**

## Paso 1. Crear el diagrama ER del proyecto

Crear el diagrama ER del proyecto y añadirlo al repositorio en el directorio `docs/uml`.

> **Nota:** este diagrama de BD debe realizarse entre todos los integrantes; solo un miembro del SQUAD se encarga de subirlo al repositorio.

1. Cree la rama `feature/erd`.
2. Adicione el modelo en la carpeta correspondiente en el repositorio.
3. Realice un PR desde la rama `feature/erd` a `develop`.
4. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

> **NOTA:** Los pasos siguientes se realizan de manera independiente en ramas diferentes y al combinar tendrán el modelo completo de datos.

---

## Paso 2. Agregar dependencias Maven al proyecto principal

En el archivo `pom.xml` del proyecto principal deben agregar las dependencias necesarias para trabajar con JPA, PostgreSQL y H2.

**Dependencias mínimas requeridas**

```xml
<dependencies>

    <!-- Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- PostgreSQL -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- H2 para pruebas -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- Validaciones -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <!-- Testing -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

</dependencies>
```

**Lo que deben hacer:**

1. Cree la rama `feature/database-config`.
2. Realice un PR desde la rama `feature/database-config` a `develop`.
3. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

---

## Paso 3. Identificar las entidades del dominio que serán persistidas

A partir del proyecto actual, cada equipo debe seleccionar **al menos 3 entidades** del dominio que deban almacenarse en base de datos.

> **NOTA:** cada equipo deberá tomar entidades que cubran las funcionalidades:
>
> - a. Autenticación.
> - b. Trabajadores (CRUD).

**Ejemplo** (de aquí en adelante se utilizará un ejemplo, pero deberá adaptarlo de acuerdo a los requerimientos del proyecto):

- **Proyecto de ejemplo E-commerce:** entidades seleccionadas: `Producto`, `Categoría` y `Pedido`.

**Lo que deben hacer:**

1. Cree la rama `feature/entities-selection`.
2. Elegir al menos 3 entidades.
3. Definir relaciones entre ellas.
4. Escribir en el `README.md` cuáles entidades seleccionaron y la justificación.
5. Realice un PR desde la rama `feature/entities-selection` a `develop`.
6. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

---

## Paso 4. Convertir las clases del dominio en entidades JPA

Cada entidad debe anotarse correctamente. Ejemplo: `Producto`

```java
package com.ejemplo.proyecto.entity;

import jakarta.persistence.*;
import jakarta.validation.constraints.NotBlank;
import java.math.BigDecimal;

@Entity
@Table(name = "productos")
public class Producto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @NotBlank
    @Column(nullable = false, length = 100)
    private String nombre;

    @Column(length = 250)
    private String descripcion;

    @Column(nullable = false)
    private BigDecimal precio;

    public Producto() {
    }

    public Producto(String nombre, String descripcion, BigDecimal precio) {
        this.nombre = nombre;
        this.descripcion = descripcion;
        this.precio = precio;
    }

    public Long getId() {
        return id;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public String getDescripcion() {
        return descripcion;
    }

    public void setDescripcion(String descripcion) {
        this.descripcion = descripcion;
    }

    public BigDecimal getPrecio() {
        return precio;
    }

    public void setPrecio(BigDecimal precio) {
        this.precio = precio;
    }
}
```

**Lo que deben hacer:**

1. Cree la rama `feature/entities`.
2. Cree cada entidad seleccionada en el paquete `/entity`.
3. Agregar la etiqueta `@Entity` a cada entidad.
4. Definir `@Table` (cuando las tablas y la clase no tengan el mismo nombre).
5. Agregar un atributo `id` a cada entidad.
6. Marcar la llave primaria con `@Id`.
7. Definir la estrategia de generación con `@GeneratedValue`. **Regla de negocio:** Id autogenerado e incremental.
8. Configurar columnas con `@Column`.
9. Agregar constructor vacío.
10. Agregar getters y setters.
11. Realice un PR desde la rama `feature/entities` a `develop`.
12. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

---

## Paso 5. Modelar relaciones entre entidades

Deben mapear **al menos una relación real del dominio**. Ejemplo: muchos productos pertenecen a una categoría.

**Categoría**

```java
@Entity
@Table(name = "categorias")
public class Categoria {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String nombre;

    public Categoria() {}

    public Categoria(String nombre) {
        this.nombre = nombre;
    }

    public Long getId() {
        return id;
    }

    public String getNombre() {
        return nombre;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }
}
```

**Producto con relación `ManyToOne`**

```java
@ManyToOne
@JoinColumn(name = "categoria_id", nullable = false)
private Categoria categoria;
```

**Relaciones válidas que pueden usar**

- `@ManyToOne`
- `@OneToMany`
- `@OneToOne`
- `@ManyToMany`

**Lo que deben hacer:**

1. Cree la rama `feature/relationships`.
2. Identificar la cardinalidad correcta.
3. Agregar la anotación correspondiente.
4. Definir la llave foránea con `@JoinColumn` cuando aplique.
5. Realice un PR desde la rama `feature/relationships` a `develop`.
6. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

---

## Paso 6. Crear los repositorios JPA

Por cada entidad principal deben crear una interfaz en el paquete `repository`. Ejemplo:

```java
package com.ejemplo.proyecto.repository;

import com.ejemplo.proyecto.entity.Producto;
import org.springframework.data.jpa.repository.JpaRepository;
import java.util.List;

public interface ProductoRepository extends JpaRepository<Producto, Long> {

    List<Producto> findByNombreContainingIgnoreCase(String nombre);
}
```

**Lo que deben hacer:**

1. Cree la rama `feature/repositories`.
2. Crear una interfaz por entidad.
3. Extender de `JpaRepository<Entidad, TipoId>`.
4. Agregar al menos una consulta derivada por nombre.
5. Realice un PR desde la rama `feature/repositories` a `develop`.
6. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

---

## Paso 7. Crear la base de datos PostgreSQL

Pueden hacerlo por Docker.

```bash
docker pull postgres

docker run --name postgres-lab8 \
  -e POSTGRES_DB=lab8db \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 5432:5432 \
  -d postgres
```

> Ajustar al nombre adecuado para el proyecto.

---

## Paso 8. Configurar PostgreSQL en el proyecto principal

Crear o modificar el archivo `src/main/resources/application.properties`.

**Configuración ejemplo**

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/lab8db
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.datasource.driver-class-name=org.postgresql.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
```

**Lo que deben hacer:**

1. Cree la rama `feature/database-setup`.
2. Crear una base de datos en PostgreSQL.
3. Ajustar nombre, usuario y contraseña.
4. Configurar `ddl-auto=update` para desarrollo.
5. Ejecutar la aplicación.
6. Realice un PR desde la rama `feature/database-setup` a `develop`.
7. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

**Verificación**

Cuando la aplicación arranque:

- Debe conectarse a PostgreSQL.
- Debe crear o actualizar tablas automáticamente.
- No debe lanzar errores de conexión.

---

## Paso 9. Crear mappers para pasar de entidades a modelos

La capa de servicios y controladores no deberían manejar entidades, sino modelos. Para ello debemos mapear cada entidad al modelo correspondiente usando la librería **MapStruct**.

**Lo que deben hacer:**

1. Cree la rama `feature/mapstruct`.
2. Cree el paquete `mapper` al mismo nivel de `entity` y `model`:

```
├── entity/      # Entidades JPA (Base de datos)
├── mapper/      # Mappers Entidades a Modelo
└── model/       # Entidades centrales del negocio
```

3. Adicione las dependencias necesarias en el `pom.xml`:

```xml
<properties>
    <mapstruct.version>1.6.3</mapstruct.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>${mapstruct.version}</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.13.0</version>
            <configuration>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${mapstruct.version}</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

4. En la carpeta `/mapper`, cree las interfaces que realizarán el mapeo de entidades a modelos y de modelos a entidades. Ejemplo:

```java
package com.ejemplo.proyecto.mapper;

import com.ejemplo.proyecto.model.ProductoModel;
import com.ejemplo.proyecto.entity.Producto;
import org.mapstruct.Mapper;

@Mapper(componentModel = "spring")
public interface ProductoMapper {

    ProductoModel toModel(Producto entity);

    Producto toEntity(ProductoModel model);
}
```

> **Nota:** al llamarse igual la entidad y el modelo, deben renombrar los modelos. Se sugiere adicionar la palabra `Model` al final de cada uno. Ejemplo: `UserModel`.

5. Realice un PR desde la rama `feature/mapstruct` a `develop`.
6. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

---

## Paso 10. Crear la capa de servicio usando los repositorios

El proyecto **no debe acceder al repositorio directamente desde el controlador**. Deben usar una capa de servicio y modelos. Ejemplo:

```java
package com.ejemplo.proyecto.service;

import com.ejemplo.proyecto.mapper.ProductoMapper;
import com.ejemplo.proyecto.model.ProductoModel;
import com.ejemplo.proyecto.repository.ProductoRepository;
import org.springframework.stereotype.Service;

@Service
public class ProductoService {

    private final ProductoRepository productoRepository;
    private final ProductoMapper productoMapper;

    public ProductoService(ProductoRepository productoRepository,
                           ProductoMapper productoMapper) {
        this.productoRepository = productoRepository;
        this.productoMapper = productoMapper;
    }

    public ProductoModel guardar(ProductoModel producto) {
        return productoMapper.toModel(
                productoRepository.save(
                        productoMapper.toEntity(producto)));
    }

    public ProductoModel buscarPorId(Long id) {
        return productoRepository.findById(id)
                .map(productoMapper::toModel)
                .orElseThrow(() -> new RuntimeException("Producto no encontrado"));
    }
}
```

**Lo que deben hacer:**

1. Cree la rama `feature/services`.
2. Actualizar los servicios creados en el Laboratorio 6, pero ahora usando persistencia.
3. Inyectar el repositorio por constructor.
4. Inyectar el mapper por constructor.
5. Mover la lógica de persistencia al servicio. No olvide mapear de modelo a entidad y de entidad a modelo usando los mappers.
6. Realice un PR desde la rama `feature/services` a `develop`.
7. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

---

## Paso 11. Ajustar los controladores para usar la base de datos

Los controladores actuales deben dejar de trabajar con listas en memoria y usar los servicios. Ejemplo:

```java
@RestController
@RequestMapping("/productos")
public class ProductoController {

    private final ProductoService productoService;

    public ProductoController(ProductoService productoService) {
        this.productoService = productoService;
    }

    @PostMapping
    public ProductoModel guardar(@RequestBody ProductoModel producto) {
        return productoService.guardar(producto);
    }

    @GetMapping
    public List<ProductoModel> listar() {
        return productoService.listar();
    }

    @GetMapping("/{id}")
    public ProductoModel buscarPorId(@PathVariable Long id) {
        return productoService.buscarPorId(id);
    }
}
```

**Lo que deben hacer** (*si no los tiene así desde el Laboratorio 6*):

1. Cree la rama `feature/controllers`.
2. Actualizar los controladores creados en el Laboratorio 6, pero ahora usando los servicios.
3. Inyectar los servicios por constructor.
4. Realice un PR desde la rama `feature/controllers` a `develop`.
5. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

**Resultado esperado:** al consumir la API, los datos deben persistir en PostgreSQL.

---

## Paso 12. Configurar H2 para pruebas

Crear el archivo `src/test/resources/application-test.properties`.

**Configuración ejemplo**

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
```

**Lo que deben hacer:**

1. Cree la rama `feature/h2-setup`.
2. Crear este archivo de configuración.
3. Asegurarse de que las pruebas usen el perfil `test`.
4. Realice un PR desde la rama `feature/h2-setup` a `develop`.
5. El PR debe ser revisado y aprobado por alguno de los integrantes del equipo, diferente a quien lo generó.

---

## Paso 13. Crear pruebas de repositorio

Cada equipo debe crear pruebas para verificar que JPA funciona correctamente. Ejemplo:

```java
package com.ejemplo.proyecto.repository;

import com.ejemplo.proyecto.entity.Producto;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.ActiveProfiles;

import java.math.BigDecimal;
import java.util.List;

import static org.junit.jupiter.api.Assertions.*;

@DataJpaTest
@ActiveProfiles("test")
class ProductoRepositoryTest {

    @Autowired
    private ProductoRepository productoRepository;

    @Test
    @DisplayName("Debe guardar un producto correctamente")
    void shouldSaveProduct() {
        Producto producto = new Producto("Teclado", "Teclado mecánico", new BigDecimal("150000"));
        Producto guardado = productoRepository.save(producto);

        assertNotNull(guardado.getId());
        assertEquals("Teclado", guardado.getNombre());
    }

    @Test
    @DisplayName("Debe buscar productos por nombre")
    void shouldFindByNombreContainingIgnoreCase() {
        productoRepository.save(new Producto("Mouse gamer", "Mouse USB", new BigDecimal("80000")));
        productoRepository.save(new Producto("Mouse inalámbrico", "Mouse BT", new BigDecimal("90000")));

        List<Producto> resultados = productoRepository.findByNombreContainingIgnoreCase("mouse");

        assertEquals(2, resultados.size());
    }
}
```

**Mínimo requerido.** Deben crear por lo menos:

1. Una prueba de guardado.
2. Una prueba de consulta.
3. Una prueba de relación entre entidades.
4. Una prueba de eliminación o actualización.

---

## Paso 14. Verificar que las pruebas corren con H2

Ejecutar:

```bash
mvn test
```

**Resultado esperado:** las pruebas deben ejecutarse usando H2, **sin depender de PostgreSQL**.

---

# PARTE B – MICROSERVICIO DE IMÁGENES CON MONGODB

## Paso 15. Crear un nuevo proyecto Spring Boot con Maven

Deben crear un **segundo repositorio**, independiente del proyecto principal.

El objetivo de este microservicio será la **gestión de todas las imágenes del proyecto**. Inicialmente modifíquelo para que funcione para las imágenes del torneo, pero tenga presente que este servicio le funcionará para imágenes de los jugadores, entre otras cosas.

**Nombre sugerido:** `image-service`

**Dependencias requeridas.** En su `pom.xml` deben incluir al menos:

```xml
<dependencies>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-mongodb</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>

</dependencies>
```

---

## Paso 16. Crear la estructura del microservicio

Estructura sugerida:

```
src/main/java/com/ejemplo/imageservice
│
├── controller
├── service
├── repository
├── model
│   └── document
└── config
```

---

## Paso 17. Configurar la conexión a MongoDB

Archivo: `src/main/resources/application.properties`

**Configuración ejemplo**

```properties
spring.data.mongodb.uri=mongodb://localhost:27017/lab8images
server.port=8081
```

**Opción Docker para MongoDB**

```bash
docker run --name mongo-lab8 -p 27017:27017 -d mongo
```

---

## Paso 18. Crear el documento Mongo para almacenar imágenes

```java
package com.ejemplo.imageservice.model.document;

import org.springframework.data.annotation.Id;
import org.springframework.data.mongodb.core.mapping.Document;

import java.time.LocalDateTime;

@Document(collection = "imagenes")
public class ImagenDocument {

    @Id
    private String id;

    private String nombre;
    private String tipoContenido;
    private Long tamano;
    private byte[] datos;
    private LocalDateTime fechaCarga;
    private String referenciaExterna;

    public ImagenDocument() {
    }

    public ImagenDocument(String nombre, String tipoContenido, Long tamano,
                          byte[] datos, LocalDateTime fechaCarga, String referenciaExterna) {
        this.nombre = nombre;
        this.tipoContenido = tipoContenido;
        this.tamano = tamano;
        this.datos = datos;
        this.fechaCarga = fechaCarga;
        this.referenciaExterna = referenciaExterna;
    }

    public String getId() {
        return id;
    }

    public String getNombre() {
        return nombre;
    }

    public String getTipoContenido() {
        return tipoContenido;
    }

    public Long getTamano() {
        return tamano;
    }

    public byte[] getDatos() {
        return datos;
    }

    public LocalDateTime getFechaCarga() {
        return fechaCarga;
    }

    public String getReferenciaExterna() {
        return referenciaExterna;
    }

    public void setNombre(String nombre) {
        this.nombre = nombre;
    }

    public void setTipoContenido(String tipoContenido) {
        this.tipoContenido = tipoContenido;
    }

    public void setTamano(Long tamano) {
        this.tamano = tamano;
    }

    public void setDatos(byte[] datos) {
        this.datos = datos;
    }

    public void setFechaCarga(LocalDateTime fechaCarga) {
        this.fechaCarga = fechaCarga;
    }

    public void setReferenciaExterna(String referenciaExterna) {
        this.referenciaExterna = referenciaExterna;
    }
}
```

---

## Paso 19. Crear el repositorio Mongo

```java
package com.ejemplo.imageservice.repository;

import com.ejemplo.imageservice.model.document.ImagenDocument;
import org.springframework.data.mongodb.repository.MongoRepository;

import java.util.List;

public interface ImagenRepository extends MongoRepository<ImagenDocument, String> {

    List<ImagenDocument> findByReferenciaExterna(String referenciaExterna);
}
```

---

## Paso 20. Crear el servicio para almacenar imágenes

```java
package com.ejemplo.imageservice.service;

import com.ejemplo.imageservice.model.document.ImagenDocument;
import com.ejemplo.imageservice.repository.ImagenRepository;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.time.LocalDateTime;
import java.util.List;

@Service
public class ImagenService {

    private final ImagenRepository imagenRepository;

    public ImagenService(ImagenRepository imagenRepository) {
        this.imagenRepository = imagenRepository;
    }

    public ImagenDocument guardar(MultipartFile archivo, String referenciaExterna) throws IOException {
        ImagenDocument imagen = new ImagenDocument(
                archivo.getOriginalFilename(),
                archivo.getContentType(),
                archivo.getSize(),
                archivo.getBytes(),
                LocalDateTime.now(),
                referenciaExterna
        );

        return imagenRepository.save(imagen);
    }

    public ImagenDocument buscarPorId(String id) {
        return imagenRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("Imagen no encontrada"));
    }

    public List<ImagenDocument> listar() {
        return imagenRepository.findAll();
    }

    public List<ImagenDocument> listarPorReferencia(String referenciaExterna) {
        return imagenRepository.findByReferenciaExterna(referenciaExterna);
    }

    public void eliminar(String id) {
        imagenRepository.deleteById(id);
    }
}
```

---

## Paso 21. Crear el controlador REST del microservicio

```java
package com.ejemplo.imageservice.controller;

import com.ejemplo.imageservice.model.document.ImagenDocument;
import com.ejemplo.imageservice.service.ImagenService;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;
import java.util.List;

@RestController
@RequestMapping("/imagenes")
public class ImagenController {

    private final ImagenService imagenService;

    public ImagenController(ImagenService imagenService) {
        this.imagenService = imagenService;
    }

    @PostMapping(consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
    public ImagenDocument subirImagen(@RequestParam("archivo") MultipartFile archivo,
                                      @RequestParam("referenciaExterna") String referenciaExterna)
            throws IOException {
        return imagenService.guardar(archivo, referenciaExterna);
    }

    @GetMapping
    public List<ImagenDocument> listar() {
        return imagenService.listar();
    }

    @GetMapping("/{id}")
    public ResponseEntity<byte[]> obtenerImagen(@PathVariable String id) {
        ImagenDocument imagen = imagenService.buscarPorId(id);

        return ResponseEntity.ok()
                .header(HttpHeaders.CONTENT_DISPOSITION,
                        "inline; filename=\"" + imagen.getNombre() + "\"")
                .contentType(MediaType.parseMediaType(imagen.getTipoContenido()))
                .body(imagen.getDatos());
    }

    @GetMapping("/referencia/{referenciaExterna}")
    public List<ImagenDocument> listarPorReferencia(@PathVariable String referenciaExterna) {
        return imagenService.listarPorReferencia(referenciaExterna);
    }

    @DeleteMapping("/{id}")
    public void eliminar(@PathVariable String id) {
        imagenService.eliminar(id);
    }
}
```

### Endpoints del microservicio

| Método   | Endpoint                                | Descripción                              |
|----------|-----------------------------------------|------------------------------------------|
| `POST`   | `/imagenes`                             | Subir una imagen (`multipart/form-data`) |
| `GET`    | `/imagenes`                             | Listar todas las imágenes                |
| `GET`    | `/imagenes/{id}`                        | Consultar una imagen por id              |
| `GET`    | `/imagenes/referencia/{referenciaExterna}` | Listar imágenes por referencia externa |
| `DELETE` | `/imagenes/{id}`                        | Eliminar una imagen                      |

---

## Paso 22. Probar el microservicio

Deben validar con **Postman, Insomnia o Swagger** los siguientes casos mínimos:

1. Subir una imagen.
2. Listar imágenes.
3. Consultar una imagen por id.
4. Eliminar una imagen.
5. Listar imágenes por referencia externa.

---

## 6. Requisitos mínimos del laboratorio

### Parte A – JPA

Cada equipo debe entregar:

- [ ] Mínimo 3 entidades JPA.
- [ ] Mínimo 3 repositorios.
- [ ] Mínimo 1 relación entre entidades.
- [ ] Conexión funcional a PostgreSQL.
- [ ] Pruebas con H2.
- [ ] Integración de persistencia con controladores y servicios.

### Parte B – MongoDB

Cada equipo debe entregar:

- [ ] Segundo repositorio independiente.
- [ ] Microservicio en Spring Boot.
- [ ] Conexión funcional a MongoDB.
- [ ] Documento para imagen.
- [ ] Repositorio Mongo.
- [ ] Endpoints REST funcionales.

---

## 7. Evidencias a entregar

1. **Enlace del repositorio principal:** `<URL del repositorio>`
2. **Enlace del repositorio del microservicio:** `<URL del repositorio>`
3. **Captura o evidencia de tablas creadas en PostgreSQL** (en el `README.md` del repositorio del proyecto):

   <!-- ![Tablas en PostgreSQL](docs/images/postgres-tablas.png) -->

4. **Evidencia de pruebas ejecutadas con H2** (en el `README.md` del repositorio del proyecto):

   <!-- ![Pruebas con H2](docs/images/pruebas-h2.png) -->

5. **Evidencia del almacenamiento de imágenes en MongoDB** (en el `README.md` del segundo repositorio):

   <!-- ![Imágenes en MongoDB](docs/images/mongodb-imagenes.png) -->

> **Nota:** los puntos 3 y 4 van en el README del repositorio principal y el punto 5 en el README del repositorio del microservicio.
