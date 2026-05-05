# 07 — `App.tsx` y `main.tsx` (wiring final)

Aquí se monta TODO. Si te equivocas en el orden de los providers, el contexto no llega o tira error.

## `main.tsx` (no lo toques — ya viene)

```tsx
// src/main.tsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import './index.css';
import App from './App.tsx';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

> **No metas providers ni router aquí.** Todo va dentro de `App.tsx` para que sea reutilizable y testeable.

## `App.tsx` — Examen 1 (IncidTech)

```tsx
// src/App.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import { IncidentsProvider } from './context/IncidentsContext';

import Navbar from './components/Navbar';
import ProtectedRoute from './components/ProtectedRoute';

import HomePage from './pages/HomePage';
import IncidentsPage from './pages/IncidentsPage';
import LoginPage from './pages/LoginPage';
import NewIncidentPage from './pages/NewIncidentPage';
import NotFoundPage from './pages/NotFoundPage';

export default function App() {
  return (
    <AuthProvider>
      <IncidentsProvider>
        <BrowserRouter>
          <Navbar />
          <main className="main-content">
            <Routes>
              <Route path="/" element={<HomePage />} />
              <Route path="/incidents" element={<IncidentsPage />} />
              <Route path="/login" element={<LoginPage />} />
              <Route
                path="/new-incident"
                element={
                  <ProtectedRoute>
                    <NewIncidentPage />
                  </ProtectedRoute>
                }
              />
              <Route path="*" element={<NotFoundPage />} />
            </Routes>
          </main>
        </BrowserRouter>
      </IncidentsProvider>
    </AuthProvider>
  );
}
```

## `App.tsx` — Examen 2 (PS4 Nexus)

```tsx
// src/App.tsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import { AuthProvider } from './context/AuthContext';
import { GamesProvider } from './context/GamesContext';
import { FavoritesProvider } from './context/FavoritesContext';

import Navbar from './components/Navbar';
import Footer from './components/Footer';
import ProtectedRoute from './components/ProtectedRoute';
import ErrorBoundary from './components/ErrorBoundary';

import HomePage from './pages/HomePage';
import LoginPage from './pages/LoginPage';
import GamesPage from './pages/GamesPage';
import GameDetailPage from './pages/GameDetailPage';
import FavoritesPage from './pages/FavoritesPage';
import NotFoundPage from './pages/NotFoundPage';

export default function App() {
  return (
    <AuthProvider>
      <GamesProvider>
        <FavoritesProvider>
          <BrowserRouter>
            <Navbar />
            <main className="main-content">
              <Routes>
                <Route path="/" element={<HomePage />} />
                <Route path="/login" element={<LoginPage />} />

                {/* Listado público de juegos, con ErrorBoundary */}
                <Route
                  path="/games"
                  element={
                    <ErrorBoundary>
                      <GamesPage />
                    </ErrorBoundary>
                  }
                />

                {/* Detalle de juego — protegido */}
                <Route
                  path="/games/:id"
                  element={
                    <ProtectedRoute>
                      <ErrorBoundary>
                        <GameDetailPage />
                      </ErrorBoundary>
                    </ProtectedRoute>
                  }
                />

                {/* Favoritos — protegido */}
                <Route
                  path="/favorites"
                  element={
                    <ProtectedRoute>
                      <FavoritesPage />
                    </ProtectedRoute>
                  }
                />

                <Route path="*" element={<NotFoundPage />} />
              </Routes>
            </main>
            <Footer />
          </BrowserRouter>
        </FavoritesProvider>
      </GamesProvider>
    </AuthProvider>
  );
}
```

## Reglas del orden de providers

1. **`AuthProvider` siempre PRIMERO** (más externo).
2. Los contextos que **dependen del usuario** (Favorites usa `useAuth`) van DESPUÉS de `AuthProvider`.
3. **`BrowserRouter` va dentro de los providers** — así puedes usar `useAuth()` en componentes que tienen `useNavigate`.
4. **`Navbar` y `Footer` van fuera de `<Routes>`** pero dentro de `<BrowserRouter>` — así se ven en todas las páginas y pueden usar `<Link>`.

```tsx
// ✅ Orden correcto
<AuthProvider>
  <FavoritesProvider> {/* depende de auth → más adentro */}
    <BrowserRouter>
      <Navbar />
      <Routes>...</Routes>
      <Footer />
    </BrowserRouter>
  </FavoritesProvider>
</AuthProvider>

// ❌ Mal: BrowserRouter fuera → no puedes usar Link en Navbar
<BrowserRouter>
  <AuthProvider>
    <Navbar />  {/* ❌ rompe si Navbar usa Link */}
  </AuthProvider>
</BrowserRouter>
```

## `HomePage` (página pública mínima)

```tsx
// src/pages/HomePage.tsx
import { Link } from 'react-router-dom';

export default function HomePage() {
  return (
    <section className="hero">
      <h1 className="hero-title">Bienvenido a IncidTech</h1>
      <p className="hero-subtitle">
        Sistema de gestión de incidencias del departamento de IT.
        Consulta las incidencias abiertas o crea una nueva.
      </p>
      <Link to="/incidents" className="btn btn-primary">
        Ver incidencias
      </Link>
    </section>
  );
}
```

## `NotFoundPage` (ruta `*`)

```tsx
// src/pages/NotFoundPage.tsx
import { Link } from 'react-router-dom';

export default function NotFoundPage() {
  return (
    <section className="not-found">
      <h1>404</h1>
      <p>Página no encontrada</p>
      <Link to="/" className="btn btn-primary">Volver al inicio</Link>
    </section>
  );
}
```

## `NewIncidentPage` (página protegida que envuelve un formulario)

```tsx
// src/pages/NewIncidentPage.tsx
import IncidentForm from '../components/IncidentForm';

export default function NewIncidentPage() {
  return (
    <section className="new-incident-page">
      <h1>Nueva incidencia</h1>
      <IncidentForm />
    </section>
  );
}
```

> **Importante**: el `<ProtectedRoute>` lo aplicas en `App.tsx` cuando defines la ruta, NO dentro del componente página.

## `.env` que debes tener (ya viene)

```bash
# .env
VITE_API_URL=http://192.168.50.120:1495/api
```

> **No cambies la IP del backend.** Si hay problemas de conexión, avisa al profe.

## Errores frecuentes

| ❌ Mal | ✅ Bien |
|---|---|
| `<BrowserRouter>` fuera de los providers | Dentro: providers → router → navbar + routes |
| `<Navbar>` dentro de cada página | Una vez en `App.tsx`, fuera de `<Routes>` |
| Olvidar la ruta `*` para 404 | Última `<Route>` siempre con `path="*"` |
| Importar `BrowserRouter` de `react-router` | De `react-router-dom` (con `-dom`) |
| `<Routes>` directamente como children del Provider | Va dentro de `<BrowserRouter>` |
| `ProtectedRoute` dentro del componente página | Va en `App.tsx`, envolviendo el `element` |
