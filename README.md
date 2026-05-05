# Resumenes examen React 2026 — Alvaro MP

Resumen práctico de todo lo que el profe pide. Está partido en ficheros para que en el examen abras solo lo que necesites.

## Orden de lectura recomendado

| # | Fichero | Qué cubre |
|---|---|---|
| 00 | [00-estructura-y-orden.md](./00-estructura-y-orden.md) | Estructura de carpetas + orden en el que escribir los ficheros |
| 01 | [01-services-api.md](./01-services-api.md) | `services/api.ts` — patrón fetch + token Bearer |
| 02 | [02-auth-context.md](./02-auth-context.md) | `AuthContext` + `useAuth` con localStorage y `getMe` |
| 03 | [03-resource-context.md](./03-resource-context.md) | Patrón CRUD genérico (Incidents / Games / Favorites…) |
| 04 | [04-protected-route-navbar.md](./04-protected-route-navbar.md) | `ProtectedRoute` y `Navbar` con renderizado condicional |
| 05 | [05-login-clasico.md](./05-login-clasico.md) | LoginPage con `useState` + `onSubmit` (Examen 1) |
| 06 | [06-login-useActionState.md](./06-login-useActionState.md) | LoginPage con `useActionState` — **OBLIGATORIO en Examen 2** |
| 07 | [07-app-wiring.md](./07-app-wiring.md) | `App.tsx` y `main.tsx` — Router + Providers + rutas |
| 08 | [08-errores-comunes.md](./08-errores-comunes.md) | Bugs que ya están en la solución parcial — NO los repitas |

## Resumen de un vistazo

- **Capa de servicios** → `fetch` + `async/await` + helper `getHeaders()` que lee token de localStorage.
- **Contextos** → 1 por dominio (`AuthContext`, y luego uno por recurso). Cada uno expone su `useXxx()`.
- **Auth** → token en localStorage, `useEffect([])` con `checkAuth → getMe()`, `loading: true` al inicio.
- **Recursos** → CRUD con actualización local optimista (`setItems(prev => [...])`) en lugar de re-fetch.
- **Routing** → `<BrowserRouter>` dentro de los providers; `ProtectedRoute` envuelve rutas privadas.
- **Forms** → Examen 1: `useState + onSubmit`. Examen 2: `useActionState` **obligatorio**.

## Reglas de oro

1. **Tipa todo desde `src/types/index.ts`** — no inventes tipos nuevos salvo que sean estados internos del contexto.
2. **El `loading` inicial del `AuthContext` es `true`** para evitar redirigir a `/login` antes de comprobar el token.
3. **El token se guarda con clave literal `'token'`** — `localStorage.setItem('token', token)`. Nunca `localStorage.setItem(token)` ni `localStorage.getItem(token)`.
4. **En operaciones autenticadas, el token va en `Authorization: Bearer <token>`** — el helper `getHeaders()` ya lo añade.
5. **`useEffect(() => { fetchX() }, [])` solo para carga inicial.** Tras crear/borrar, actualiza estado local.
6. **`ProtectedRoute` debe esperar a `loading === false` antes de redirigir** — si no, hay flash y redirige aunque el usuario sí esté logueado.
7. **`Toaster` (sonner) es opcional y NO viene en el examen** — usa mensajes de error/carga en pantalla con `useState`.

## Archivos clave en el repositorio del curso (referencia)

- **Patrón canónico CRUD**: `CursoReact2526-main/sesion-08-repaso-final/crud-gestion-empresarial-bd/frontend/src/`
- **Ejemplo `useActionState`**: `CursoReact2526-main/sesion-07-forms-react19/src/pages/formBasico/FormTodoList.tsx`
- **Login clásico**: `CursoReact2526-t-main/sesion-08-repaso/.../components/auth/LoginForm.tsx`
- **Wiring de App con Outlet**: `CursoReact2526-t-main/sesion-08-repaso/.../apps/App3Professional.tsx`
