# Proyecto Integrador · Sistema de Reporte Ciudadano

**Curso:** Desarrollo de Aplicaciones Web (DAB-2201) · TecNM Cancún  
**Equipo:** Pedro Pablo Pool Ché (`XxZoroxX10`) y Armando Chim Dominguez (`BrokeniGS`)  
**Dominio:** Tablero → Tarea, adaptado a reportes ciudadano.
**Repositorio:** https://github.com/XxZoroxX10/Proyecto-Integrador_ReporteCiudadano_DesarrolloWeb  
**Docente:** MCC. Iván Márquez Larios (`ijmarquezl`)

---

## 1. ¿Qué resuelve?

Los vecinos reportan problemas de su colonia (baches, alumbrado, basura) y les dan seguimiento hasta que se resuelven. Cada **tablero** agrupa los reportes de una zona y cada **tarea** es un reporte con categoría, ubicación y estado.

## 2. Alcance

- Entidad principal: `Tablero` (zona o colonia).
- Entidad secundaria: `Tarea` (reporte ciudadano), relación 1:N.
- Usuario: registro, login y rol `user`/`admin` (Hito 2).
- Interfaz: una SPA con login, lista, formulario y eliminar (Hito 4).
- Fuera de alcance: subir fotos, mapas, notificaciones y APIs externas.

## 3. Stack

- Backend: FastAPI + Pydantic v2
- Seguridad: bcrypt (hash de contraseñas) + PyJWT (HS256)
- Base de datos: en memoria (Hitos 1 y 2); SQLite desde el Hito 3
- ORM / migraciones: SQLAlchemy + Alembic (Hito 3)
- Frontend: HTML, CSS y JavaScript vanilla
- Pruebas: pytest + TestClient (httpx)

## 4. Cómo ejecutarlo

```bash
python -m venv .venv
source .venv/bin/activate       # En Windows: .venv\Scripts\activate
pip install -r requirements.txt
export JWT_SECRET_KEY="una-clave-larga-y-aleatoria"   # opcional en desarrollo
export ADMIN_PASSWORD="elige-una-contraseña"           # crea el usuario admin al arrancar
uvicorn src.main:app --reload --port 8000
```

Abre `http://127.0.0.1:8000/docs`.

| Variable | Para qué | Si no se define |
| :--- | :--- | :--- |
| `JWT_SECRET_KEY` | Firma de los tokens | Se genera una aleatoria al arrancar (los tokens dejan de servir al reiniciar) |
| `JWT_EXPIRA_MINUTOS` | Vigencia del token | 30 |
| `ADMIN_USUARIO` / `ADMIN_PASSWORD` | Administrador inicial | No se crea ningún admin |
| `BCRYPT_COSTO` | Costo de bcrypt | 12 |

### Estructura

```text
src/
├── main.py          # arma la app: routers, manejadores de error y admin inicial
├── core/            # config, seguridad (bcrypt + JWT) y dependencias (Depends)
├── models/          # entidades internas: Usuario, Tablero, Tarea
├── schemas/         # DTOs de entrada y salida (Pydantic)
├── repositories/    # acceso a datos (en memoria; no importa FastAPI)
├── services/        # reglas de negocio y de propiedad
└── routers/         # rutas HTTP: auth, reportes y páginas HTML
```

### Pruebas

```bash
pytest -q
```

Resultado actual: **18 passed** (8 del contrato del Hito 1 + las 10 del Hito 2).

## 5. Endpoints (Hito 2)

| Método | Ruta | Acceso | Respuesta |
| :--- | :--- | :--- | :--- |
| GET | `/health` | Público | 200 `{"status":"ok"}` |
| POST | `/api/registro` | Público | 201 `{"id","usuario","rol":"user"}` / 409 / 422 |
| POST | `/api/token` | Público | 200 `{"access_token","token_type","expires_in"}` / 401 |
| GET | `/api/perfil` | Con token | 200 / 401 |
| GET | `/api/tableros` | Con token | 200 `{"tableros":[...]}`: un `user` ve los suyos; un `admin`, todos |
| POST | `/api/tableros` | Con token | 201 (el dueño es quien lo crea) / 422 |
| GET | `/api/tableros/{id}/tareas` | Dueño o admin | 200 `{"tareas":[...]}` / 403 / 404 |
| POST | `/api/tableros/{id}/tareas` | Dueño o admin | 201 / 403 / 404 / 422 |
| GET | `/api/tareas/{id}` | Dueño o admin | 200 / 403 / 404 |
| DELETE | `/api/tareas/{id}` | Dueño o admin | 204 / 403 / 404 |

Sin token o con token inválido → **401** con `WWW-Authenticate: Bearer`; con token pero sin permiso → **403**.  
Errores con sobre uniforme: `{"error": {"codigo": "...", "mensaje": "..."}}` (`NO_AUTENTICADO`, `SIN_PERMISO`, `RECURSO_NO_ENCONTRADO`, `METODO_NO_PERMITIDO`, `CONFLICTO`, `DATOS_INVALIDOS`).

Ejemplo:

```bash
curl -X POST http://127.0.0.1:8000/api/registro -H "Content-Type: application/json" \
  -d '{"usuario":"ana","password":"secreta123"}'
TOKEN=$(curl -s -X POST http://127.0.0.1:8000/api/token -H "Content-Type: application/json" \
  -d '{"usuario":"ana","password":"secreta123"}' | python -c "import sys,json;print(json.load(sys.stdin)['access_token'])")
curl -X POST http://127.0.0.1:8000/api/tableros -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" -d '{"nombre":"Colonia Centro"}'
# 201 {"id":1,"nombre":"Colonia Centro","descripcion":null,"usuario_id":1}
```

## 6. Hitos y etiquetas

| Hito | Sesión | Tag | Estado |
| :--- | :---: | :--- | :--- |
| 1. Contrato y mock | 10 | `v0.1-hito1` | [x] Código y 8 pruebas |
| 2. Capas y seguridad | 20 | `v0.2-hito2` | [x] Capas, JWT, roles y 10 pruebas |
| 3. Persistencia y OWASP | 30 | `v0.3-hito3` | [ ] |
| 4. Producto | 40 | `v1.0-final` | [ ] |

## 7. Documentos

- [Propuesta](docs/propuesta.md)
- [Acuerdo de equipo](docs/equipo.md)
- [Decisiones por hito](docs/decisiones.md)
- [Checkpoint 1](docs/checkpoint-01.md)

## 8. IA y autoría

- [x] Las consultas a IA están documentadas en `docs/decisiones.md`.
- [x] La interfaz (`templates/`) proviene de un proyecto previo del equipo y está declarada en `docs/decisiones.md`.
- [x] No hay secretos ni datos personales en el repositorio.
