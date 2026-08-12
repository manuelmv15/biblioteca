# Sistema de Control de Biblioteca

Control de acceso para biblioteca universitaria con 16 PCs (Universidad de El Salvador, Facultad Multidisciplinaria de Oriente). Registra sesiones de estudiantes, sincroniza datos al servidor central y genera reportes.

Este repositorio es el **índice del proyecto**: documenta la arquitectura y enlaza a los dos repos donde vive el código. No contiene código propio ni se despliega — cada componente se clona e instala desde su propio repositorio.

## Repositorios

| Repo | Corre en | Contenido |
|---|---|---|
| [biblioteca_servidor](https://github.com/manuelmv15/biblioteca_servidor) | PC maestra | Backend FastAPI + panel web admin |
| [biblioteca_cliente](https://github.com/manuelmv15/biblioteca_cliente) | 16 PCs hijas | App de escritorio PyQt6 en modo kiosko |

## Arquitectura

```
PC maestra  →  FastAPI + SQLite  →  Panel web admin
PC hija ×16 →  PyQt6 kiosko     →  SQLite local + sync automático
```

El kiosko funciona **sin internet** — guarda sesiones localmente y las sincroniza cuando recupera conexión.

---

## Requisitos

- Python 3.10+
- Red local entre PCs (o internet vía Cloudflare)
- Linux o Windows en cualquier PC

---

## Flujo completo

```
Estudiante llega a una PC hija
  ↓
Ingresa carnet → app busca en caché local
  ↓ (si no existe en caché y hay internet)
Consulta al servidor → guarda en caché local
  ↓ (si no existe en ningún lado)
Pantalla de registro → llena formulario → guarda local + envía al servidor
  ↓
Login exitoso → pantalla con nombre, carrera y timer en vivo
  ↓
"Cerrar sesión" → guarda hora_fin en SQLite local (sincronizado = 0)
  ↓
Cada 30 segundos → hilo en segundo plano detecta conexión
  ↓
Envía sesiones pendientes al servidor → marca sincronizado = 1
```

---

## Contrato servidor ↔ cliente

API que expone `biblioteca_servidor` y consume `biblioteca_cliente`. Si cambia, hay que coordinar un cambio en ambos repos.

| Método | Ruta | Descripción |
|---|---|---|
| `POST` | `/auth/login` | Login admin, retorna JWT |
| `POST` | `/sync` | Recibe lote de sesiones desde PC hija |
| `POST` | `/estudiantes` | Registra nuevo estudiante |
| `GET` | `/estudiantes/{carnet}` | Busca estudiante por carnet |
| `GET` | `/reportes/sesiones` | Lista sesiones con filtros opcionales |
| `GET` | `/reportes/pcs-activas` | PCs con conexión en últimos 5 min |
| `GET` | `/reportes/resumen-dia` | Totales del día |
| `GET` | `/health` | Health check (usado por clientes para detectar conexión) |

Parámetros de `/reportes/sesiones`: `fecha`, `pc_id`, `carnet`, `carrera`, `limit`, `offset`

---

## Orden de despliegue

```
1. Clonar biblioteca_servidor → instalar en la PC maestra
2. Anotar IP local de la PC maestra
3. Clonar biblioteca_cliente → en cada PC hija:
      pip install -r requirements.txt
      python setup.py
4. Probar con 2-3 PCs antes de las 16
5. Verificar en el panel admin que llegan las sesiones
6. (Opcional) túnel Cloudflare para acceso externo — ver biblioteca_servidor
```

Instalación, configuración, autostart y actualización de cada componente: ver el README de su propio repo (arriba).

---

## Historia previa

Antes de este índice, cliente y servidor vivían como ramas (`cliente`, `servidor`) de este mismo repositorio. Esas ramas se conservan aquí como archivo histórico, congeladas — el desarrollo activo de cada una sigue en su repo correspondiente.
