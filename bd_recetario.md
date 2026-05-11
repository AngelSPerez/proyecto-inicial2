# 📖 Recetario Digital — Base de Datos

> Sistema de gestión de recetas digitales: usuarios, recetas, ingredientes, pasos, etiquetas y favoritos.

---

## 📋 Índice

- [Descripción del proyecto](#descripción-del-proyecto)
- [Entidades y atributos](#entidades-y-atributos)
- [Diagrama Entidad-Relación](#diagrama-entidad-relación)
- [Script SQL](#script-sql)
- [Relaciones clave](#relaciones-clave)

---

## Descripción del proyecto

**Recetario Digital** es una aplicación de gestión de recetas de cocina que permite a los usuarios crear, organizar, buscar y compartir recetas. El sistema administra ingredientes, pasos de preparación, categorías, etiquetas, comentarios y listas de favoritos.

---

## Entidades y atributos

### 👤 Usuario

| Atributo         | Tipo           | Restricción         | Descripción                        |
|------------------|----------------|---------------------|------------------------------------|
| id_usuario       | INT            | PK, AUTO_INCREMENT  | Identificador único                |
| nombre           | VARCHAR(100)   | NOT NULL            | Nombre completo                    |
| username         | VARCHAR(50)    | UNIQUE, NOT NULL    | Nombre de usuario                  |
| email            | VARCHAR(150)   | UNIQUE, NOT NULL    | Correo electrónico                 |
| password_hash    | VARCHAR(255)   | NOT NULL            | Contraseña encriptada              |
| avatar_url       | VARCHAR(300)   | NULL                | URL de imagen de perfil            |
| bio              | TEXT           | NULL                | Descripción del usuario            |
| activo           | BOOLEAN        | DEFAULT TRUE        | Estado de la cuenta                |
| fecha_registro   | DATETIME       | DEFAULT NOW()       | Fecha de creación de cuenta        |
| rol              | ENUM           | DEFAULT 'usuario'   | usuario / editor / admin           |

---

### 🍽️ Receta

| Atributo          | Tipo           | Restricción         | Descripción                        |
|-------------------|----------------|---------------------|------------------------------------|
| id_receta         | INT            | PK, AUTO_INCREMENT  | Identificador único                |
| id_usuario        | INT            | FK → Usuario        | Autor de la receta                 |
| id_categoria      | INT            | FK → Categoria      | Categoría principal                |
| titulo            | VARCHAR(200)   | NOT NULL            | Nombre de la receta                |
| descripcion       | TEXT           | NULL                | Descripción breve                  |
| imagen_url        | VARCHAR(300)   | NULL                | Foto de portada                    |
| tiempo_prep_min   | INT            | NULL                | Tiempo de preparación (minutos)    |
| tiempo_coccion_min| INT            | NULL                | Tiempo de cocción (minutos)        |
| porciones         | INT            | NOT NULL            | Número de porciones                |
| dificultad        | ENUM           | NOT NULL            | facil / media / dificil            |
| tipo_cocina       | VARCHAR(80)    | NULL                | Mexicana, Italiana, etc.           |
| calorias_aprox    | INT            | NULL                | Calorías por porción               |
| es_publica        | BOOLEAN        | DEFAULT TRUE        | Visibilidad de la receta           |
| estado            | ENUM           | DEFAULT 'borrador'  | borrador / publicada / archivada   |
| fecha_creacion    | DATETIME       | DEFAULT NOW()       | Fecha de publicación               |
| fecha_actualizacion| DATETIME      | ON UPDATE NOW()     | Última modificación                |

---

### 🗂️ Categoria

| Atributo      | Tipo         | Restricción        | Descripción                         |
|---------------|--------------|--------------------|-------------------------------------|
| id_categoria  | INT          | PK, AUTO_INCREMENT | Identificador único                 |
| nombre        | VARCHAR(100) | UNIQUE, NOT NULL   | Nombre de la categoría              |
| descripcion   | TEXT         | NULL               | Detalle de la categoría             |
| icono_url     | VARCHAR(300) | NULL               | Ícono representativo                |
| activa        | BOOLEAN      | DEFAULT TRUE       | Estado de la categoría              |

> **Ejemplos:** Desayunos, Sopas, Postres, Bebidas, Ensaladas, Carnes, Vegano, Sin gluten

---

### 🥕 Ingrediente

| Atributo        | Tipo         | Restricción        | Descripción                         |
|-----------------|--------------|--------------------|-------------------------------------|
| id_ingrediente  | INT          | PK, AUTO_INCREMENT | Identificador único                 |
| nombre          | VARCHAR(150) | UNIQUE, NOT NULL   | Nombre del ingrediente              |
| descripcion     | TEXT         | NULL               | Notas o sustituciones               |
| calorias_por_100g| DECIMAL(6,2)| NULL               | Información nutricional             |
| imagen_url      | VARCHAR(300) | NULL               | Foto del ingrediente                |
| activo          | BOOLEAN      | DEFAULT TRUE       | Disponibilidad en el catálogo       |

---

### 🔗 RecetaIngrediente *(tabla pivote)*

| Atributo        | Tipo          | Restricción              | Descripción                       |
|-----------------|---------------|--------------------------|-----------------------------------|
| id              | INT           | PK, AUTO_INCREMENT       | Identificador único               |
| id_receta       | INT           | FK → Receta              | Receta a la que pertenece         |
| id_ingrediente  | INT           | FK → Ingrediente         | Ingrediente utilizado             |
| id_unidad       | INT           | FK → UnidadMedida        | Unidad de medida                  |
| cantidad        | DECIMAL(8,3)  | NOT NULL                 | Cantidad necesaria                |
| notas           | VARCHAR(200)  | NULL                     | Ej: "picado fino", "al gusto"     |
| es_opcional     | BOOLEAN       | DEFAULT FALSE            | Si el ingrediente es opcional     |
| orden           | INT           | DEFAULT 0                | Orden en la lista de ingredientes |

---

### 👣 PasoReceta

| Atributo        | Tipo          | Restricción        | Descripción                         |
|-----------------|---------------|--------------------|-------------------------------------|
| id_paso         | INT           | PK, AUTO_INCREMENT | Identificador único                 |
| id_receta       | INT           | FK → Receta        | Receta a la que pertenece           |
| numero_paso     | INT           | NOT NULL           | Orden del paso (1, 2, 3…)           |
| instruccion     | TEXT          | NOT NULL           | Descripción del paso                |
| imagen_url      | VARCHAR(300)  | NULL               | Foto ilustrativa del paso           |
| duracion_min    | INT           | NULL               | Tiempo estimado del paso            |
| consejo         | VARCHAR(300)  | NULL               | Tip adicional del chef              |

---

### 🏷️ Etiqueta

| Atributo     | Tipo         | Restricción        | Descripción                          |
|--------------|--------------|--------------------|--------------------------------------|
| id_etiqueta  | INT          | PK, AUTO_INCREMENT | Identificador único                  |
| nombre       | VARCHAR(80)  | UNIQUE, NOT NULL   | Nombre de la etiqueta                |
| color_hex    | CHAR(7)      | NULL               | Color para UI (ej: `#FF5733`)        |

> **Ejemplos:** #rápido, #5ingredientes, #sinlactosa, #navidad, #picante, #niños

---

### 🔗 RecetaEtiqueta *(tabla pivote)*

| Atributo    | Tipo | Restricción        | Descripción                     |
|-------------|------|--------------------|----------------------------------|
| id_receta   | INT  | FK → Receta        | Receta etiquetada               |
| id_etiqueta | INT  | FK → Etiqueta      | Etiqueta aplicada               |

> **PK compuesta:** (id_receta, id_etiqueta)

---

### 💬 Comentario

| Atributo        | Tipo         | Restricción        | Descripción                         |
|-----------------|--------------|--------------------|-------------------------------------|
| id_comentario   | INT          | PK, AUTO_INCREMENT | Identificador único                 |
| id_receta       | INT          | FK → Receta        | Receta comentada                    |
| id_usuario      | INT          | FK → Usuario       | Autor del comentario                |
| id_padre        | INT          | FK → Comentario    | NULL = comentario raíz, o respuesta |
| contenido       | TEXT         | NOT NULL           | Texto del comentario                |
| calificacion    | TINYINT      | NULL (1–5)         | Estrellas (solo comentarios raíz)   |
| fecha           | DATETIME     | DEFAULT NOW()      | Fecha y hora                        |
| editado         | BOOLEAN      | DEFAULT FALSE      | Si fue modificado                   |

---

### ❤️ Favorito

| Atributo   | Tipo     | Restricción        | Descripción                     |
|------------|----------|--------------------|---------------------------------|
| id_usuario | INT      | FK → Usuario       | Usuario que guarda              |
| id_receta  | INT      | FK → Receta        | Receta guardada                 |
| fecha      | DATETIME | DEFAULT NOW()      | Fecha en que se marcó favorito  |

> **PK compuesta:** (id_usuario, id_receta)

---

### 📐 UnidadMedida

| Atributo    | Tipo        | Restricción        | Descripción                          |
|-------------|-------------|--------------------|--------------------------------------|
| id_unidad   | INT         | PK, AUTO_INCREMENT | Identificador único                  |
| nombre      | VARCHAR(50) | UNIQUE, NOT NULL   | Nombre completo (ej: "kilogramo")    |
| abreviatura | VARCHAR(15) | UNIQUE, NOT NULL   | Abreviatura (ej: "kg", "tza", "pza") |
| tipo        | ENUM        | NOT NULL           | peso / volumen / pieza / otro        |

---

## Diagrama Entidad-Relación

```
┌───────────────┐        ┌─────────────────────┐        ┌──────────────────┐
│   CATEGORIA   │        │       RECETA         │        │     USUARIO      │
│───────────────│        │─────────────────────│        │──────────────────│
│ PK id_categ.  │──1──N──│ PK id_receta         │──N──1──│ PK id_usuario    │
│    nombre     │        │ FK id_usuario         │        │    username      │
│    descripcion│        │ FK id_categoria       │        │    email         │
│    icono_url  │        │    titulo             │        │    password_hash │
│    activa     │        │    descripcion        │        │    avatar_url    │
└───────────────┘        │    imagen_url         │        │    rol           │
                         │    tiempo_prep_min    │        └──────────────────┘
                         │    tiempo_coccion_min │                │
                         │    porciones          │                │ 1
                         │    dificultad         │                │
                         │    es_publica         │                N
                         │    estado             │        ┌──────────────────┐
                         └─────────────────────┘        │   COMENTARIO     │
                                   │                     │──────────────────│
                    ┌──────────────┼──────────────┐      │ PK id_comentario │
                    │              │              │      │ FK id_receta     │
                    1              1              1      │ FK id_usuario    │
                    │              │              │      │ FK id_padre(self)│
                    N              N              N      │    contenido     │
                    │              │              │      │    calificacion  │
          ┌─────────────┐  ┌──────────────┐  ┌──────────┐    fecha       │
          │RECETAINGRED.│  │  PASORECETA  │  │RECETAETIQ│  └──────────────────┘
          │─────────────│  │──────────────│  │──────────│
          │ PK id        │  │ PK id_paso   │  │FK id_rec.│  ┌──────────────────┐
          │ FK id_receta │  │ FK id_receta │  │FK id_etiq│  │    FAVORITO      │
          │ FK id_ingred.│  │ numero_paso  │  └──────────┘  │──────────────────│
          │ FK id_unidad │  │ instruccion  │       │         │ FK id_usuario    │
          │ cantidad     │  │ imagen_url   │       N         │ FK id_receta     │
          │ notas        │  │ duracion_min │       │         │    fecha         │
          │ es_opcional  │  │ consejo      │  ┌──────────┐  └──────────────────┘
          └─────────────┘  └──────────────┘  │ ETIQUETA │
                 │                            │──────────│
         ┌───────┴──────┐                    │PK id_etiq│
         │              │                    │   nombre │
         N              N                    │color_hex │
         │              │                    └──────────┘
┌────────────┐  ┌─────────────┐
│INGREDIENTE │  │ UNIDADMEDIDA│
│────────────│  │─────────────│
│PK id_ingred│  │PK id_unidad │
│   nombre   │  │   nombre    │
│descripcion │  │abreviatura  │
│calorias/100│  │   tipo      │
│imagen_url  │  └─────────────┘
└────────────┘
```

---

## Relaciones clave

```
Categoria        1 ──── N   Receta
Usuario          1 ──── N   Receta
Usuario          1 ──── N   Comentario
Usuario          N ──── M   Receta          (via Favorito)
Receta           1 ──── N   PasoReceta
Receta           N ──── M   Ingrediente     (via RecetaIngrediente)
Receta           N ──── M   Etiqueta        (via RecetaEtiqueta)
Receta           1 ──── N   Comentario
Comentario       1 ──── N   Comentario      (respuestas, auto-referencia)
UnidadMedida     1 ──── N   RecetaIngrediente
```

---

## Script SQL

```sql
-- ============================================================
--  RECETARIO DIGITAL — Script de creación de base de datos
-- ============================================================

CREATE DATABASE IF NOT EXISTS recetario_digital
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

USE recetario_digital;

-- ------------------------------------------------------------
-- 1. UNIDAD DE MEDIDA
-- ------------------------------------------------------------
CREATE TABLE UnidadMedida (
  id_unidad    INT          AUTO_INCREMENT PRIMARY KEY,
  nombre       VARCHAR(50)  NOT NULL UNIQUE,
  abreviatura  VARCHAR(15)  NOT NULL UNIQUE,
  tipo         ENUM('peso','volumen','pieza','otro') NOT NULL
);

-- ------------------------------------------------------------
-- 2. CATEGORIA
-- ------------------------------------------------------------
CREATE TABLE Categoria (
  id_categoria  INT          AUTO_INCREMENT PRIMARY KEY,
  nombre        VARCHAR(100) NOT NULL UNIQUE,
  descripcion   TEXT,
  icono_url     VARCHAR(300),
  activa        BOOLEAN      NOT NULL DEFAULT TRUE
);

-- ------------------------------------------------------------
-- 3. USUARIO
-- ------------------------------------------------------------
CREATE TABLE Usuario (
  id_usuario      INT          AUTO_INCREMENT PRIMARY KEY,
  nombre          VARCHAR(100) NOT NULL,
  username        VARCHAR(50)  NOT NULL UNIQUE,
  email           VARCHAR(150) NOT NULL UNIQUE,
  password_hash   VARCHAR(255) NOT NULL,
  avatar_url      VARCHAR(300),
  bio             TEXT,
  activo          BOOLEAN      NOT NULL DEFAULT TRUE,
  fecha_registro  DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
  rol             ENUM('usuario','editor','admin') NOT NULL DEFAULT 'usuario'
);

-- ------------------------------------------------------------
-- 4. INGREDIENTE
-- ------------------------------------------------------------
CREATE TABLE Ingrediente (
  id_ingrediente    INT           AUTO_INCREMENT PRIMARY KEY,
  nombre            VARCHAR(150)  NOT NULL UNIQUE,
  descripcion       TEXT,
  calorias_por_100g DECIMAL(6,2),
  imagen_url        VARCHAR(300),
  activo            BOOLEAN       NOT NULL DEFAULT TRUE
);

-- ------------------------------------------------------------
-- 5. ETIQUETA
-- ------------------------------------------------------------
CREATE TABLE Etiqueta (
  id_etiqueta  INT         AUTO_INCREMENT PRIMARY KEY,
  nombre       VARCHAR(80) NOT NULL UNIQUE,
  color_hex    CHAR(7)
);

-- ------------------------------------------------------------
-- 6. RECETA
-- ------------------------------------------------------------
CREATE TABLE Receta (
  id_receta            INT          AUTO_INCREMENT PRIMARY KEY,
  id_usuario           INT          NOT NULL,
  id_categoria         INT          NOT NULL,
  titulo               VARCHAR(200) NOT NULL,
  descripcion          TEXT,
  imagen_url           VARCHAR(300),
  tiempo_prep_min      INT,
  tiempo_coccion_min   INT,
  porciones            INT          NOT NULL,
  dificultad           ENUM('facil','media','dificil') NOT NULL,
  tipo_cocina          VARCHAR(80),
  calorias_aprox       INT,
  es_publica           BOOLEAN      NOT NULL DEFAULT TRUE,
  estado               ENUM('borrador','publicada','archivada') NOT NULL DEFAULT 'borrador',
  fecha_creacion       DATETIME     NOT NULL DEFAULT CURRENT_TIMESTAMP,
  fecha_actualizacion  DATETIME     ON UPDATE CURRENT_TIMESTAMP,
  CONSTRAINT fk_receta_usuario   FOREIGN KEY (id_usuario)   REFERENCES Usuario(id_usuario),
  CONSTRAINT fk_receta_categoria FOREIGN KEY (id_categoria) REFERENCES Categoria(id_categoria)
);

-- ------------------------------------------------------------
-- 7. RECETA-INGREDIENTE (tabla pivote)
-- ------------------------------------------------------------
CREATE TABLE RecetaIngrediente (
  id             INT           AUTO_INCREMENT PRIMARY KEY,
  id_receta      INT           NOT NULL,
  id_ingrediente INT           NOT NULL,
  id_unidad      INT           NOT NULL,
  cantidad       DECIMAL(8,3)  NOT NULL,
  notas          VARCHAR(200),
  es_opcional    BOOLEAN       NOT NULL DEFAULT FALSE,
  orden          INT           NOT NULL DEFAULT 0,
  CONSTRAINT fk_ri_receta      FOREIGN KEY (id_receta)      REFERENCES Receta(id_receta),
  CONSTRAINT fk_ri_ingrediente FOREIGN KEY (id_ingrediente) REFERENCES Ingrediente(id_ingrediente),
  CONSTRAINT fk_ri_unidad      FOREIGN KEY (id_unidad)      REFERENCES UnidadMedida(id_unidad)
);

-- ------------------------------------------------------------
-- 8. PASO RECETA
-- ------------------------------------------------------------
CREATE TABLE PasoReceta (
  id_paso      INT  AUTO_INCREMENT PRIMARY KEY,
  id_receta    INT  NOT NULL,
  numero_paso  INT  NOT NULL,
  instruccion  TEXT NOT NULL,
  imagen_url   VARCHAR(300),
  duracion_min INT,
  consejo      VARCHAR(300),
  CONSTRAINT fk_paso_receta FOREIGN KEY (id_receta) REFERENCES Receta(id_receta),
  UNIQUE KEY uq_paso (id_receta, numero_paso)
);

-- ------------------------------------------------------------
-- 9. RECETA-ETIQUETA (tabla pivote)
-- ------------------------------------------------------------
CREATE TABLE RecetaEtiqueta (
  id_receta   INT NOT NULL,
  id_etiqueta INT NOT NULL,
  PRIMARY KEY (id_receta, id_etiqueta),
  CONSTRAINT fk_re_receta   FOREIGN KEY (id_receta)   REFERENCES Receta(id_receta),
  CONSTRAINT fk_re_etiqueta FOREIGN KEY (id_etiqueta) REFERENCES Etiqueta(id_etiqueta)
);

-- ------------------------------------------------------------
-- 10. COMENTARIO
-- ------------------------------------------------------------
CREATE TABLE Comentario (
  id_comentario  INT      AUTO_INCREMENT PRIMARY KEY,
  id_receta      INT      NOT NULL,
  id_usuario     INT      NOT NULL,
  id_padre       INT      DEFAULT NULL,
  contenido      TEXT     NOT NULL,
  calificacion   TINYINT  CHECK (calificacion BETWEEN 1 AND 5),
  fecha          DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  editado        BOOLEAN  NOT NULL DEFAULT FALSE,
  CONSTRAINT fk_com_receta  FOREIGN KEY (id_receta)  REFERENCES Receta(id_receta),
  CONSTRAINT fk_com_usuario FOREIGN KEY (id_usuario) REFERENCES Usuario(id_usuario),
  CONSTRAINT fk_com_padre   FOREIGN KEY (id_padre)   REFERENCES Comentario(id_comentario)
);

-- ------------------------------------------------------------
-- 11. FAVORITO
-- ------------------------------------------------------------
CREATE TABLE Favorito (
  id_usuario  INT      NOT NULL,
  id_receta   INT      NOT NULL,
  fecha       DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (id_usuario, id_receta),
  CONSTRAINT fk_fav_usuario FOREIGN KEY (id_usuario) REFERENCES Usuario(id_usuario),
  CONSTRAINT fk_fav_receta  FOREIGN KEY (id_receta)  REFERENCES Receta(id_receta)
);

-- ============================================================
-- DATOS DE EJEMPLO — Catálogos base
-- ============================================================

INSERT INTO UnidadMedida (nombre, abreviatura, tipo) VALUES
  ('kilogramo',    'kg',   'peso'),
  ('gramo',        'g',    'peso'),
  ('litro',        'l',    'volumen'),
  ('mililitro',    'ml',   'volumen'),
  ('taza',         'tza',  'volumen'),
  ('cucharada',    'cda',  'volumen'),
  ('cucharadita',  'cdta', 'volumen'),
  ('pieza',        'pza',  'pieza'),
  ('diente',       'dnt',  'pieza'),
  ('al gusto',     'c/n',  'otro');

INSERT INTO Categoria (nombre, descripcion) VALUES
  ('Desayunos',   'Recetas para comenzar el día'),
  ('Sopas',       'Caldos, cremas y consomés'),
  ('Platos fuertes', 'Carnes, aves y mariscos'),
  ('Ensaladas',   'Opciones frescas y ligeras'),
  ('Postres',     'Dulces, pasteles y helados'),
  ('Bebidas',     'Aguas, jugos y cocteles'),
  ('Vegano',      'Sin productos de origen animal'),
  ('Sin gluten',  'Apto para celíacos');
```

---

## Tecnologías sugeridas

| Capa          | Opción A         | Opción B         |
|---------------|------------------|------------------|
| Base de datos | MySQL 8          | PostgreSQL 15    |
| Backend       | Node.js + Express| Python + FastAPI |
| ORM           | Sequelize        | SQLAlchemy       |
| Frontend      | React + Vite     | Next.js          |
| Autenticación | JWT + bcrypt     | OAuth 2.0        |
| Almacenamiento| Cloudinary       | AWS S3           |

---

*Proyecto: Recetario Digital | Versión: 1.0 | Motor: MySQL 8 / MariaDB 10.6+*
