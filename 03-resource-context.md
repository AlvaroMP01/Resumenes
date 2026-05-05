# 03 — Contexto de un recurso (CRUD genérico)

Sirve para `IncidentsContext`, `GamesContext`, `FavoritesContext`, `CompaniesContext`… Todos siguen el **mismo patrón**.

## Lo que tiene que exponer

| Campo | Tipo | Descripción |
|---|---|---|
| `items` | `Item[]` | Lista del recurso (`incidents`, `games`, `favorites`…) |
| `loading` | `boolean` | Cargando lista |
| `error` | `string \| null` *(opcional)* | Error de la última operación |
| `fetchItems()` | `() => Promise<void>` | Recarga la lista del backend |
| `addItem(data)` | `(data) => Promise<void>` | Crea un item |
| `removeItem(id)` | `(id) => Promise<void>` | Elimina un item |

> **Mira tu enunciado**: a veces se llaman `addIncident`/`removeIncident`, otras `createGame`/`deleteGame`. **Usa los nombres EXACTOS del PDF**.

## Plantilla canónica — IncidentsContext (Examen 1)

```tsx
// src/context/IncidentsContext.tsx
import { createContext, useContext, useEffect, useState, type ReactNode } from 'react';
import { incidentsAPI } from '../services/api';
import type { Incident, CreateIncidentDTO } from '../types';

export interface IncidentsContextType {
  incidents: Incident[];
  loading: boolean;
  fetchIncidents: () => Promise<void>;
  addIncident: (data: CreateIncidentDTO) => Promise<void>;
  removeIncident: (id: number) => Promise<void>;
}

const IncidentsContext = createContext<IncidentsContextType | undefined>(undefined);

export function IncidentsProvider({ children }: { children: ReactNode }) {
  const [incidents, setIncidents] = useState<Incident[]>([]);
  const [loading, setLoading] = useState<boolean>(true);

  // LEER — pide la lista al backend
  async function fetchIncidents(): Promise<void> {
    try {
      setLoading(true);
      const response = await incidentsAPI.getIncidents();
      // Adapta según el formato real:
      //   - Si la respuesta es { ok, data: Incident[] } → response.data ?? []
      //   - Si la respuesta es Incident[] directamente → response
      setIncidents(response.data ?? []);
    } catch (err) {
      console.error('Error al cargar incidencias:', err);
      setIncidents([]);
    } finally {
      setLoading(false);
    }
  }

  // CREAR — envía al backend y añade al inicio del array local
  async function addIncident(data: CreateIncidentDTO): Promise<void> {
    const response = await incidentsAPI.createIncident(data);
    const created = response.data;
    if (!created) return;
    // [nuevo, ...anteriores] → aparece arriba en la lista
    setIncidents((prev) => [created, ...prev]);
  }

  // ELIMINAR — envía al backend y filtra el array local
  async function removeIncident(id: number): Promise<void> {
    await incidentsAPI.deleteIncident(id);
    setIncidents((prev) => prev.filter((inc) => inc.id !== id));
  }

  // Carga inicial al montar
  useEffect(() => {
    fetchIncidents();
  }, []);

  const value: IncidentsContextType = {
    incidents,
    loading,
    fetchIncidents,
    addIncident,
    removeIncident,
  };

  return (
    <IncidentsContext.Provider value={value}>
      {children}
    </IncidentsContext.Provider>
  );
}

export function useIncidents(): IncidentsContextType {
  const context = useContext(IncidentsContext);
  if (context === undefined) {
    throw new Error('useIncidents debe usarse dentro de IncidentsProvider');
  }
  return context;
}
```

## Variante con búsqueda — GamesContext (Examen 2)

```tsx
// src/context/GamesContext.tsx
import { createContext, useContext, useEffect, useState, type ReactNode } from 'react';
import { gamesAPI } from '../services/api';
import type { Game } from '../types';
import { useAuth } from './AuthContext';

export interface GamesContextType {
  games: Game[];
  loading: boolean;
  error: string | null;
  fetchGames: (search?: string) => Promise<void>;
  deleteById: (id: number) => Promise<void>;
}

const GamesContext = createContext<GamesContextType | undefined>(undefined);

export function GamesProvider({ children }: { children: ReactNode }) {
  const [games, setGames] = useState<Game[]>([]);
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);
  const { token } = useAuth();

  async function fetchGames(search?: string): Promise<void> {
    try {
      setLoading(true);
      setError(null);
      const response = await gamesAPI.getGames(1, 24, search ?? '');
      setGames(response.games);
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Error al cargar juegos';
      setError(message);
      setGames([]);
    } finally {
      setLoading(false);
    }
  }

  async function deleteById(id: number): Promise<void> {
    if (!token) return; // sin sesión, no se permite eliminar
    await gamesAPI.deleteGame(id);
    setGames((prev) => prev.filter((g) => g.id !== id));
  }

  useEffect(() => {
    fetchGames();
  }, []);

  const value: GamesContextType = { games, loading, error, fetchGames, deleteById };
  return <GamesContext.Provider value={value}>{children}</GamesContext.Provider>;
}

export function useGames(): GamesContextType {
  const context = useContext(GamesContext);
  if (!context) throw new Error('useGames debe usarse dentro de GamesProvider');
  return context;
}
```

## Variante con dependencia de auth — FavoritesContext (Examen 2)

Los favoritos cambian cuando cambia el usuario logueado:

```tsx
// src/context/FavoritesContext.tsx
import { createContext, useContext, useEffect, useState, type ReactNode } from 'react';
import { favoritesAPI } from '../services/api';
import type { Game } from '../types';
import { useAuth } from './AuthContext';

export interface FavoritesContextType {
  favorites: Game[];
  loading: boolean;
  toggleFavorite: (gameId: number) => Promise<void>;
  isFavorite: (gameId: number) => boolean;
}

const FavoritesContext = createContext<FavoritesContextType | undefined>(undefined);

export function FavoritesProvider({ children }: { children: ReactNode }) {
  const [favorites, setFavorites] = useState<Game[]>([]);
  const [loading, setLoading] = useState<boolean>(false);
  const { token } = useAuth();

  // Recarga favoritos cada vez que cambia el token (login/logout)
  useEffect(() => {
    async function load() {
      if (!token) {
        setFavorites([]);
        return;
      }
      try {
        setLoading(true);
        const data = await favoritesAPI.getFavorites();
        setFavorites(data);
      } catch {
        setFavorites([]);
      } finally {
        setLoading(false);
      }
    }
    load();
  }, [token]);

  function isFavorite(gameId: number): boolean {
    return favorites.some((g) => g.id === gameId);
  }

  async function toggleFavorite(gameId: number): Promise<void> {
    if (!token) return;
    if (isFavorite(gameId)) {
      await favoritesAPI.removeFavorite(gameId);
      setFavorites((prev) => prev.filter((g) => g.id !== gameId));
    } else {
      await favoritesAPI.addFavorite(gameId);
      // Refrescar para tener el objeto Game completo
      const data = await favoritesAPI.getFavorites();
      setFavorites(data);
    }
  }

  const value: FavoritesContextType = { favorites, loading, toggleFavorite, isFavorite };
  return <FavoritesContext.Provider value={value}>{children}</FavoritesContext.Provider>;
}

export function useFavorites(): FavoritesContextType {
  const context = useContext(FavoritesContext);
  if (!context) throw new Error('useFavorites debe usarse dentro de FavoritesProvider');
  return context;
}
```

## Patrones clave para recordar

```tsx
// ✅ Crear: añadir al principio del array (UX: lo nuevo se ve arriba)
setItems((prev) => [nuevo, ...prev]);

// ✅ Eliminar: filtrar por id
setItems((prev) => prev.filter((x) => x.id !== id));

// ✅ Actualizar: map y reemplazar el que coincide
setItems((prev) => prev.map((x) => (x.id === id ? actualizado : x)));

// ✅ Carga inicial automática
useEffect(() => {
  fetchItems();
}, []);

// ✅ Recarga cuando cambia algo externo (ej: el usuario)
useEffect(() => {
  fetchItems();
}, [token]); // o [user], [searchTerm], etc.
```

## Errores frecuentes

| ❌ Mal | ✅ Bien |
|---|---|
| Tras crear → `await fetchItems()` (re-pide todo) | `setItems(prev => [nuevo, ...prev])` |
| Modificar el array original (`items.push(x)`) | Crear array nuevo: `[...items, x]` |
| `useEffect(() => fetchX(), [])` con dependencia faltante | Si depende de algo, ponlo: `[token]` |
| Olvidar `setLoading(false)` en el `finally` | Siempre en `finally`, nunca solo en `try` |
| Crear tipos en este fichero | Importar de `types/index.ts` |
