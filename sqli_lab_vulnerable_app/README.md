# SQL Injection Vulnerable Lab — Solución
### ISW-1013 Calidad del Software — Universidad Técnica Nacional

---

> ✅ **Esta rama contiene la solución corregida** del laboratorio de SQL Injection.
> Las vulnerabilidades originales han sido identificadas, explotadas y corregidas.
> Rama: `GuillermoSolorzano_KasandraCruz`

---

## Inicio rápido

```bash
# 1. Clonar el repositorio
git clone https://github.com/josedavidcar/sqli_utn.git
cd sqli_utn

# 2. Cambiar a la rama de solución
git checkout GuillermoSolorzano_KasandraCruz

# 3. Levantar la aplicación
docker compose up --build

# 4. Abrir en el navegador
# http://localhost:5000
```

## Credenciales de prueba

| Usuario  | Contraseña  | Rol   |
|----------|-------------|-------|
| admin    | Admin123    | admin |
| analyst  | Analyst123  | user  |
| student  | Student123  | user  |

> ⚠️ Las contraseñas ahora se almacenan con hash (scrypt). Si la base de datos ya existía antes de aplicar la corrección V-03, elimine `db/lab.db` y vuelva a levantar con `docker compose up --build` para regenerarla.

---

## Vulnerabilidades corregidas

| ID   | Tipo                          | Ubicación                    | Estado     |
|------|-------------------------------|------------------------------|------------|
| V-01 | SQL Injection – Login         | `app.py` ruta `/login`       | ✅ Corregido |
| V-02 | SQL Injection – Búsqueda      | `app.py` ruta `/search`      | ✅ Corregido |
| V-03 | Contraseñas en texto plano    | `app.py` / `init_db()`       | ✅ Corregido |
| V-04 | SECRET_KEY hardcodeada        | `app.py` línea config        | ✅ Corregido |
| V-05 | debug=True activo             | `app.py` al final            | ✅ Corregido |
| V-06 | Exposición de query SQL       | `templates/search.html`      | ✅ Corregido |
| V-07 | Menú admin visible a todos    | `templates/layout.html`      | ✅ Corregido |
| V-08 | Sin protección CSRF           | Formularios                  | ✅ Corregido |

---

## Resumen de correcciones aplicadas

### V-01 y V-02 — SQL Injection
Se reemplazaron todas las consultas construidas con f-strings por **consultas parametrizadas** usando `?` como placeholder. SQLite se encarga de escapar los valores correctamente.

```python
# Antes (vulnerable)
query = f"SELECT ... WHERE username = '{username}'"

# Después (corregido)
user = conn.execute("SELECT ... WHERE username = ?", (username,)).fetchone()
```

### V-03 — Contraseñas en texto plano
Las contraseñas ahora se almacenan con **hash scrypt** usando `werkzeug.security`. El login verifica el hash sin comparar strings directamente.

```python
from werkzeug.security import generate_password_hash, check_password_hash

# Al crear: generate_password_hash("Admin123")
# Al validar: check_password_hash(user["password"], password_ingresado)
```

### V-04 — SECRET_KEY hardcodeada
La clave secreta se carga desde una **variable de entorno**.

```python
app.config["SECRET_KEY"] = os.environ.get("SECRET_KEY", "dev-secret-key-local")
```

### V-05 — debug=True
El modo debug se controla desde una **variable de entorno**.

```python
app.run(debug=os.environ.get("FLASK_DEBUG", "false").lower() == "true")
```

### V-06 — Query SQL expuesta
Se eliminó el bloque en `search.html` que renderizaba `raw_query` en pantalla.

### V-07 — Menú Admin visible para todos
El enlace Admin en `layout.html` ahora solo se muestra si el rol de sesión es `admin`.

```html
{% if session.get('role') == 'admin' %}
  <a href="{{ url_for('admin') }}">Admin</a>
{% endif %}
```

### V-08 — Sin protección CSRF
Se instaló `Flask-WTF` y se habilitó `CSRFProtect`. Todos los formularios incluyen el token CSRF.

```python
from flask_wtf.csrf import CSRFProtect
csrf = CSRFProtect(app)
```

---

## Estructura del proyecto

```
sqli_lab_vulnerable_app/
├── app.py                  ← Lógica principal (vulnerabilidades corregidas)
├── requirements.txt        ← Incluye flask-wtf
├── Dockerfile
├── docker-compose.yml
├── db/                     ← Base de datos SQLite (se crea automáticamente)
├── templates/
│   ├── layout.html         ← V-07 corregido
│   ├── login.html          ← V-08 corregido
│   ├── dashboard.html
│   ├── search.html         ← V-06 y V-08 corregidos
│   └── admin.html
└── static/
    └── style.css
```

## Detener la aplicación

```bash
docker compose down
```

---

## Integrantes

- Guillermo Antonio Solórzano Ochoa
- Kassandra Cruz Arroyo
