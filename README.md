# Aula Norte — Sistema de matrícula

Aplicación web en español para administrar cuentas, matrículas y notas. Utiliza Spring Boot, Spring Data JPA, PostgreSQL y una interfaz HTML adaptable a teléfonos y computadoras.

## Requisitos

- JDK 21 o superior.
- Maven 3.9 o superior.
- PostgreSQL 14 o superior.

## Base de datos

La base de datos configurada se llama `matricula`. El script conserva los registros existentes, completa el esquema y agrega datos de ejemplo sin repetirlos:

```powershell
psql -U postgres -c "CREATE DATABASE matricula;"
psql -U postgres -d matricula -f database/schema_postgres.sql
```

La aplicación también crea las tablas nuevas al iniciar por primera vez. No borres la base existente: contiene el historial académico.

La contraseña se lee desde `DB_PASSWORD` o desde el archivo local `application-secrets.properties`, que está excluido de Git. El usuario se configura con `DB_USER` y, si no se indica, se usa `postgres`.

## Iniciar y entrar

Desde la carpeta del proyecto ejecuta:

```powershell
$clave = Read-Host "Contraseña de PostgreSQL" -AsSecureString
$texto = [System.Net.NetworkCredential]::new("", $clave).Password
[System.IO.File]::WriteAllText("$PWD\application-secrets.properties", "spring.datasource.password=$texto`n", [System.Text.UTF8Encoding]::new($false))
Remove-Variable texto, clave
mvn spring-boot:run
```

Abre **http://localhost:8082** en el navegador. Si no hay cuentas, aparecerá el formulario seguro para crear el primer administrador. Usa una contraseña propia de al menos 12 caracteres. La instalación no crea contraseñas predeterminadas.

## Permisos

| Perfil | Puede hacer |
|---|---|
| Administrador | Crear cuentas de alumno y docente; crear alumnos nuevos; asignar asignaturas a docentes; registrar, editar y eliminar matrículas. No puede modificar notas. |
| Docente | Ver matrículas de las asignaturas que tiene asignadas; crear, editar y eliminar sus evaluaciones. La nota admite valores de 0 a 20. |
| Alumno | Ver y actualizar su propio nombre y correo; consultar sus matrículas y las notas publicadas. El código y la facultad no se pueden editar desde su cuenta. |

El administrador registra cuentas de alumnos y docentes desde **Cuentas**. En una cuenta de docente debe seleccionarse al menos una asignatura. Los alumnos también pueden vincularse a un registro existente.

Las contraseñas se guardan con BCrypt, nunca se muestran en el panel, y cada sesión solo puede entrar a las pantallas de su perfil. La primera configuración del administrador queda cerrada al completar el registro.

## Base de datos y capas

El proyecto mantiene cinco entidades académicas principales:

- `facultad` se relaciona con `asignatura` y `estudiante`.
- `estudiante` y `asignatura` se relacionan mediante `matricula`.
- `matricula` se relaciona con `pago`.

El portal añade `cuenta`, `asignacion_docente` y `nota` para aplicar permisos, asignar cursos y conservar cada evaluación. Las notas se eliminan junto con la matrícula; los pagos conservan las restricciones del esquema.

El código Java separa controladores, servicios, repositorios y entidades. Las clases nuevas incluyen comentarios cortos en español que describen su función y las reglas importantes.

## API REST

Cada entidad académica (`facultades`, `asignaturas`, `estudiantes`, `matriculas` y `pagos`) conserva las operaciones CRUD:

| Método | Ruta | Acción |
|---|---|---|
| `GET` | `/api/{entidad}` | Listar registros. |
| `GET` | `/api/{entidad}/{id}` | Consultar un registro. |
| `POST` | `/api/{entidad}` | Crear un registro. |
| `PUT` | `/api/{entidad}/{id}` | Actualizar un registro. |
| `DELETE` | `/api/{entidad}/{id}` | Eliminar un registro. |

`/api/notas` expone el mismo CRUD para docentes autenticados. La aplicación comprueba en cada operación que el docente tenga asignado el curso de la matrícula. Los endpoints académicos existentes requieren un administrador.

Las relaciones de los endpoints se envían por ID, por ejemplo: `facultadId`, `estudianteId`, `asignaturaId` y `matriculaId`. Los errores conservan el formato `{timestamp, status, mensaje, errores}`.

## Reportes y pruebas

Los reportes PDF de listas se pueden generar en:

```text
/api/reportes/facultades/pdf
/api/reportes/asignaturas/pdf
/api/reportes/estudiantes/pdf
/api/reportes/matriculas/pdf
/api/reportes/pagos/pdf
```

La aplicación genera reportes y datos de ejemplo desde el esquema PostgreSQL. Usa [`requests.http`](./requests.http) con la extensión REST Client; inicia sesión primero en el portal y usa el mismo usuario administrador para probar los endpoints académicos.

Ejecuta las pruebas unitarias y compila el proyecto con:

```powershell
mvn test
mvn clean package
```
