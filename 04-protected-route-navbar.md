# 04 — `ProtectedRoute` y `Navbar`

Dos componentes pequeños pero **críticos** para la nota. Si los haces mal, redirige cuando no toca o muestra cosas que no debería.

## `ProtectedRoute`

### Reglas

1. Si `loading === true` → mostrar mensaje de carga (no redirigir todavía).
2. Si `loading === false` y no hay user → `<Navigate to="/login" replace />`.
3. Si hay user → renderizar `children`.

> **El paso 1 es crítico**: si redireccionas mientras `loading` está en `true`, el usuario logueado vería el login un instante porque `checkAuth` aún no terminó.

### Versión con `children` (Examen 1 y 2)

```tsx
// src/components/ProtectedRoute.tsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';
import type { ReactNode } from 'react';

interface ProtectedRouteProps {
  children: ReactNode;
}

export default function ProtectedRoute({ children }: ProtectedRouteProps) {
  const { user, loading } = useAuth();

  if (loading) {
    return (
      <div className="loading">
        <p>Comprobando sesión…</p>
      </div>
    );
  }

  if (!user) {
    return <Navigate to="/login" replace />;
  }

  return <>{children}</>;
}
```

### Uso en `App.tsx`

```tsx
<Route
  path="/new-incident"
  element={
    <ProtectedRoute>
      <NewIncidentPage />
    </ProtectedRoute>
  }
/>
```

### Versión con `Outlet` (alternativa más limpia para muchas rutas protegidas)

```tsx
import { Navigate, Outlet } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

export default function ProtectedRoute() {
  const { user, loading } = useAuth();
  if (loading) return <p>Comprobando sesión…</p>;
  if (!user) return <Navigate to="/login" replace />;
  return <Outlet />;
}
```

Y en App.tsx:
```tsx
<Route element={<ProtectedRoute />}>
  <Route path="/games/:id" element={<GameDetailPage />} />
  <Route path="/favorites" element={<FavoritesPage />} />
</Route>
```

> Las dos formas son válidas. **Usa la primera** (con `children`) si el enunciado no dice nada — es lo que aparece en los ejemplos del profe.

## `Navbar`

### Reglas

- Visible en todas las páginas (lo monta `App.tsx`, NO cada página).
- Renderizado condicional según `user`:
  - Sin sesión → enlace a `/login`.
  - Con sesión → nombre del usuario + botón "Cerrar sesión" + enlaces a rutas privadas.

### Plantilla canónica

```tsx
// src/components/Navbar.tsx
import { Link, useNavigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

export default function Navbar() {
  const { user, logout } = useAuth();
  const navigate = useNavigate();

  function handleLogout() {
    logout();
    navigate('/');
  }

  return (
    <nav className="navbar">
      <div className="navbar-brand">
        <Link to="/">IncidTech</Link>
      </div>

      <div className="navbar-links">
        <Link to="/">Inicio</Link>
        <Link to="/incidents">Incidencias</Link>

        {/* Solo visible si hay sesión */}
        {user && <Link to="/new-incident">Nueva incidencia</Link>}

        {/* Renderizado condicional según sesión */}
        {user ? (
          <>
            <span className="navbar-user">{user.name}</span>
            <button onClick={handleLogout} className="btn btn-danger">
              Cerrar sesión
            </button>
          </>
        ) : (
          <Link to="/login">Iniciar sesión</Link>
        )}
      </div>
    </nav>
  );
}
```

### Variante para Examen 2 (con contador de favoritos)

```tsx
import { Link, useNavigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';
import { useFavorites } from '../context/FavoritesContext';

export default function Navbar() {
  const { user, logout } = useAuth();
  const { favorites } = useFavorites();
  const navigate = useNavigate();

  function handleLogout() {
    logout();
    navigate('/');
  }

  return (
    <nav className="navbar">
      <Link to="/" className="navbar-brand">PS4 Nexus</Link>

      <div className="navbar-links">
        <Link to="/">Inicio</Link>
        <Link to="/games">Juegos</Link>

        {user && (
          <Link to="/favorites">
            Favoritos {favorites.length > 0 && <span className="badge">{favorites.length}</span>}
          </Link>
        )}

        {user ? (
          <>
            <span>{user.username}</span>
            <button onClick={handleLogout} className="btn btn-danger">Cerrar sesión</button>
          </>
        ) : (
          <Link to="/login" className="btn btn-primary">Iniciar sesión</Link>
        )}
      </div>
    </nav>
  );
}
```

## `Footer` (sólo Examen 2)

```tsx
// src/components/Footer.tsx
export default function Footer() {
  return (
    <footer className="footer">
      <p>Examen de React 2026</p>
      <p>
        Tu Nombre Apellido — {' '}
        <a href="https://github.com/TU_USUARIO" target="_blank" rel="noreferrer">
          @TU_USUARIO
        </a>
      </p>
    </footer>
  );
}
```

> **Pon TU NOMBRE y TU GitHub real**. El profe lo revisa.

## Errores frecuentes

| ❌ Mal | ✅ Bien |
|---|---|
| `if (!user) return <Navigate.../>` (sin esperar `loading`) | Comprobar `loading` PRIMERO |
| `<Navigate to="/login">` sin `replace` | `<Navigate to="/login" replace />` (no añade al historial) |
| Renderizar Navbar dentro de cada página | Renderizarlo UNA vez en `App.tsx` |
| Mostrar el botón de logout siempre | Renderizado condicional con `user ? ... : ...` |
| Usar `<a href="/login">` para navegar interno | Usa `<Link to="/login">` |
| `window.location.href = '/'` tras logout | Usa `useNavigate()`: `navigate('/')` |
