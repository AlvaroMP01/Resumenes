# 00 — Estructura de carpetas y orden de implementación

## Estructura final esperada

```
src/
├── App.tsx                        ← Router + Providers + rutas
├── main.tsx                       ← (ya viene) StrictMode + createRoot
├── index.css                      ← (ya viene) NO lo toques
├── vite-env.d.ts                  ← (ya viene) NO lo toques
├── types/
│   └── index.ts                   ← (ya viene) tipos del dominio
├── services/
│   └── api.ts                     ← Apartado 1
├── context/
│   ├── AuthContext.tsx            ← Apartado 2 (incluye useAuth)
│   └── XxxContext.tsx             ← Apartado 3 (Incidents, Games, Favorites…)
├── components/
│   ├── Navbar.tsx                 ← visible en todas las páginas
│   ├── ProtectedRoute.tsx         ← envuelve rutas privadas
│   ├── ErrorBoundary.tsx          ← (en Examen 2 ya viene dado)
│   ├── XxxCard.tsx                ← muestra un item individual
│   ├── XxxForm.tsx                ← formulario para crear
│   └── XxxList.tsx                ← lista (opcional, a veces basta con map)
└── pages/
    ├── HomePage.tsx               ← pública, mensaje + botón
    ├── LoginPage.tsx              ← Examen 1: useState | Examen 2: useActionState
    ├── XxxPage.tsx                ← lista pública
    ├── NewXxxPage.tsx             ← protegida, contiene XxxForm
    └── NotFoundPage.tsx           ← ruta `*`
```

> **Nota**: el enunciado a veces dice `contexts/` (plural) y otras `context/`. Mira siempre el PDF de tu examen para usar **EXACTAMENTE** el nombre que pide. Si no lo dice, usa `context/`.

## Orden de implementación (importante para no atascarte)

Ve por capas: backend → estado → UI → routing. Si lo haces así, cuando llegues al Navbar todo lo demás ya funciona.

### 1. Tipos (5 min)
- Abre `src/types/index.ts`.
- Léelo entero, anota los nombres de interfaces y DTOs.
- Si necesitas un tipo nuevo (ej. `AuthState`), créalo dentro del fichero del contexto, NO en `types/`.

### 2. `services/api.ts` (15 min)
- Función helper `getHeaders()` que lee token de localStorage.
- Una función `async` por endpoint, devolviendo `Promise<TipoExacto>`.
- Manejo de error: `if (!response.ok) throw new Error(...)`.
- Ver [01-services-api.md](./01-services-api.md).

### 3. `context/AuthContext.tsx` + `useAuth` (25 min)
- Estado: `user`, `token`, `loading`, `error?`.
- `login`, `logout`, `checkAuth` (montaje).
- Persistencia en `localStorage` con clave `'token'`.
- Custom hook `useAuth()` que lanza error fuera del Provider.
- Ver [02-auth-context.md](./02-auth-context.md).

### 4. Contexto de recurso (20 min cada uno)
- Estado: `items`, `loading`, `error?` y funciones CRUD.
- Carga inicial con `useEffect([])`.
- Update local optimista tras `create`/`delete`.
- Custom hook `useXxx()`.
- Ver [03-resource-context.md](./03-resource-context.md).

### 5. Componentes de UI (30–40 min)
- `XxxCard`: recibe el item por prop tipada, render condicional según el rol/estado.
- `XxxForm`: formulario con `useState` o `useActionState` según el examen.
- Páginas que los componen (`XxxPage`, `NewXxxPage`, `HomePage`).
- Ver [05-login-clasico.md](./05-login-clasico.md) o [06-login-useActionState.md](./06-login-useActionState.md).

### 6. Routing (20 min)
- `Navbar` con renderizado condicional según `useAuth()`.
- `ProtectedRoute` con `loading` + `Navigate` a `/login`.
- `App.tsx` con `<BrowserRouter>` y todas las rutas.
- Ver [04-protected-route-navbar.md](./04-protected-route-navbar.md) y [07-app-wiring.md](./07-app-wiring.md).

### 7. Probar el flujo completo (15 min)
- `npm run dev`.
- Navega a `/`, prueba ir a la lista (debe funcionar sin login).
- Login → debe redirigir y mostrar tu nombre en navbar.
- Recargar la página estando logueado → debe seguir logueado.
- Crear un item → debe aparecer al instante.
- Eliminar → debe desaparecer al instante.
- Logout → token desaparece de localStorage.

## Reparto de tiempo orientativo

| Fase | Tiempo (3h examen) | Tiempo (4h examen) |
|---|---|---|
| Lectura completa enunciado + setup | 15 min | 20 min |
| services/api.ts | 15 min | 20 min |
| AuthContext + useAuth | 25 min | 30 min |
| ResourceContext(s) | 25 min (×1) | 50 min (×2) |
| Componentes y páginas | 50 min | 70 min |
| Routing + Navbar + ProtectedRoute | 25 min | 30 min |
| Probar y arreglar | 25 min | 40 min |

> **Margen de seguridad**: deja siempre 15 min al final para verificar que `npm run dev` arranca sin errores de TypeScript ni warnings rojos.
