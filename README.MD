# Base de Datos Twitter (twitter_bd)

Este repositorio contiene la estructura y datos iniciales para una base de datos similar a Twitter, diseñada con MySQL. Incluye tablas para usuarios, tweets, seguidores y me gusta en los tweets, además de un trigger para mantener el conteo de seguidores.

---

## Tabla de contenidos
- [Descripción](#descripción)
- [Estructura de la base de datos](#estructura-de-la-base-de-datos)
  - [usuarios (users)](#usuarios-users)
  - [tweets](#tweets)
  - [seguidores (followers)](#seguidores-followers)
  - [likes de Tweets (tweet_likes)](#likes-de-tweets-tweet_likes)
- [Triggers y eventos](#triggers-y-eventos)
- [Datos iniciales](#datos-iniciales)
- [Instrucciones de uso](#instrucciones-de-uso)

---

## Descripción
Esta base de datos simula una plataforma tipo Twitter, permitiendo gestionar usuarios, tweets, seguidores y Likes en los tweets. Como características adicionales, se incluye un trigger que actualiza automáticamente el conteo de seguidores de cada usuario al momento de que alguien le sigue.

---

## Estructura de la base de datos

### Usuarios (`users`)
| Columna          | Tipo            | Restricciones                         | Comentarios                                |
|------------------|-----------------|--------------------------------------|--------------------------------------------|
| user_id        | `int`          | PRIMARY KEY, AUTO_INCREMENT         | Identificador único del usuario            |
| user_handle    | `varchar(50)`  | UNIQUE                               | Nombre de usuario                        |
| email_address  | `varchar(100)` | UNIQUE                               | Correo electrónico                         |
| first_name     | `varchar(100)` |                                      | Nombre                                       |
| last_name      | `varchar(100)` |                                      | Apellido                                     |
| phonenumber    | `char(10)`     | UNIQUE, NULLABLE                     | Número de teléfono                        |
| follower_count | `int`          | DEFAULT 0                          | Número de seguidores                       |
| create_at      | `timestamp`    | DEFAULT CURRENT_TIMESTAMP            | Fecha de creación                          |

### Tweets (`tweets`)
| Columna          | Tipo             | Restricciones                         | Comentarios                                |
|------------------|------------------|--------------------------------------|--------------------------------------------|
| tweet_id       | `int`           | PRIMARY KEY, AUTO_INCREMENT         | Identificador único del tweet             |
| user_id        | `int`           | FOREIGN KEY (`users(user_id)`)      | ID del autor del tweet                    |
| tweet_text     | `varchar(250)`  | NOT NULL                            | Texto del tweet                            |
| num_likes      | `int`           | DEFAULT 0                          | Número de Likes                          |
| num_retweets   | `int`           | DEFAULT 0                          | Número de retweets                       |
| num_comments   | `int`           | DEFAULT 0                          | Número de comentarios                     |
| create_at      | `timestamp`     | DEFAULT CURRENT_TIMESTAMP            | Fecha y hora de publicación                |

### Seguidores (`followers`)
| Columna         | Tipo             | Restricciones                                                | Comentarios                                       |
|-----------------|------------------|--------------------------------------------------------------|---------------------------------------------------|
| follower_id    | `int`            | FOREIGN KEY (`users(user_id)`), CHECK (`follower_id` <> `following_id`) | ID del seguidor                                  |
| following_id   | `int`            | FOREIGN KEY (`users(user_id)`)                                | ID del usuario seguido                           |
| PRIMARY KEY    | (`follower_id`, `following_id`) |  | Clave compuesta para evitar duplicados          |

Este esquema incorpora un trigger que actualiza automáticamente el conteo de seguidores en la tabla `users` cuando se añade un nuevo seguidor.

### Likes en Tweets (`tweet_likes`)
| Columna      | Tipo             | Restricciones                         | Comentarios                               |
|--------------|------------------|--------------------------------------|-------------------------------------------|
| user_id      | `int`            | FOREIGN KEY (`users(user_id)`)      | ID del usuario que da Like               |
| tweet_id     | `int`            | FOREIGN KEY (`tweets(tweet_id)`)     | ID del tweet que recibe Like             |
| PRIMARY KEY  | (`user_id`, `tweet_id`) |                            | Clave compuesta para evitar duplicados|

### Datos de ejemplo
Se insertaron datos de prueba en todas las tablas para ilustrar la estructura y relaciones.

---

## Triggers y eventos

- **`increase_follower_count`**: Trigger que se activa después de insertar un nuevo registro en `followers`, incrementando el contador de seguidores en la tabla `users` para el usuario seguido.

```sql  
CREATE TRIGGER increase_follower_count  
AFTER INSERT ON followers  
FOR EACH ROW  
BEGIN  
    UPDATE users SET follower_count = follower_count + 1  
    WHERE user_id = NEW.following_id;  
END;  