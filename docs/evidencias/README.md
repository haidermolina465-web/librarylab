# Evidencias de ejecución de LibraryLab

Fecha de ejecución: 2026-10-15 19:30 (hora local).

Estas evidencias demuestran que la solución compila, se conecta a MySQL real,
aplica las migraciones, siembra datos y responde por HTTP según lo especificado.

## 1. MySQL

- Servidor: MySQL 26.7.0 (local), puerto **3306**.
- Directorio de datos: `C:\ProgramData\MySQL\MySQL Server 26.7\Data`.
- Base de datos: `library_lab` (utf8mb4 / utf8mb4_unicode_ci).
- Usuario del proyecto: `library_lab` / `library_lab` (con permisos sobre `library_lab`).
- Usuario administrador local: `root` / `123456789`.

La aplicación usa su **cadena de conexión por defecto**, sin variables de entorno:

```
Server=localhost;Port=3306;Database=library_lab;User=library_lab;Password=library_lab;
```

### Migración aplicada

```powershell
dotnet ef database update --project src/LibraryLab.Infrastructure --startup-project src/LibraryLab.Api
```

Resultado: `Done.` con la migración `20261003001904_InitialCreate`.

Tablas creadas: `__efmigrationshistory`, `books`, `equipment`, `loans`, `resources`, `users`.

## 2. Verificación de integración real (HTTP)

Se levantó la API en `http://localhost:5109` con `Database__InitializeOnStartup=true`
y se ejecutó:

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\Verify-LibraryLab.ps1 -ApiBaseUrl http://localhost:5109
```

Resultado: **VERIFICACION CORRECTA: 76 aserciones y 38 respuestas HTTP reales.**
Se conservan los registros con prefijo `VERIFY`.

Cubre: creación/consulta/duplicados de usuarios, libros y equipos; errores 400 y 404;
préstamo y devolución de libro y de equipo; y dos solicitudes concurrentes para el
mismo recurso (un único préstamo activo).

## 3. Estado de la base tras la verificación

| Tabla     | Registros |
|-----------|-----------|
| Users     | 2         |
| Books     | 3         |
| Equipment | 2         |
| Loans     | 3         |
| Loans activos | 0     |

## 4. Pruebas automatizadas

```
dotnet test
```

Resultado: 39 pruebas correctas, 0 con error, 0 omitidas.

- LibraryLab.Patterns.Tests: 5/5
- LibraryLab.Application.Tests: 5/5
- LibraryLab.Domain.Tests: 7/7
- LibraryLab.Infrastructure.Tests: 2/2
- LibraryLab.Architecture.Tests: 2/2
- LibraryLab.Api.Tests: 18/18

## 5. Archivos de evidencia

- `verify-librarilab.log`: salida de la verificación HTTP.
- `mysql-verificacion.log`: versión, puerto, tablas, conteos y registros VERIFY.
- `api-endpoints.log`: respuestas crudas de GET /api/users, /api/books, /api/equipment y /api/loans.
- `logs/api3306.out.log`: log de inicio de la API y siembra de datos (contra el puerto 3306).
- `logs/api3306.err.log`: log de errores de la API (vacío).

## 6. Notas sobre el servidor MySQL

El servicio de MySQL se ejecuta como **proceso manual** (no como servicio de Windows),
porque registrar un servicio requiere privilegios de administrador en este equipo.
Consecuencias:

- El servidor **no se inicia automáticamente** al reiniciar el equipo.
- Para arrancarlo manualmente:

```powershell
& "C:\Program Files\MySQL\MySQL Server 26.7\bin\mysqld.exe" `
  "--basedir=C:\Program Files\MySQL\MySQL Server 26.7" `
  "--datadir=C:\ProgramData\MySQL\MySQL Server 26.7\Data" `
  --port=3306 `
  "--log-error=C:\ProgramData\MySQL\MySQL Server 26.7\Data\err.log" `
  "--lc-messages-dir=C:\Program Files\MySQL\MySQL Server 26.7\share" `
  "--plugin-dir=C:\Program Files\MySQL\MySQL Server 26.7\lib\plugin"
```

- Para instalarlo como servicio permanente (requiere una consola de administrador):

```powershell
& "C:\Program Files\MySQL\MySQL Server 26.7\bin\mysqld.exe" --install MySQL267 `
  "--defaults-file=C:\ProgramData\MySQL\MySQL Server 26.7\my.ini"
```
