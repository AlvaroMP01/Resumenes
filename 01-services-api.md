# 01 — `services/api.ts` (capa de servicios)

Patrón **exacto** del profe. Sirve para los dos exámenes y para cualquier examen futuro con la misma arquitectura.

## Reglas

- Una sola constante `API_URL` con `import.meta.env.VITE_API_URL`.
- Un helper `getHeaders()` que añade el `Bearer <token>` si hay token en localStorage.
- Funciones `async` que devuelven `Promise<TipoExacto>` (importado de `types/`).
- Manejo de errores: si `!response.ok`, lanza `Error` con mensaje útil.
- **NO uses axios.** Solo `fetch`.

## Plantilla canónica

```ts
// src/services/api.ts
import type {
  AuthResponse,
  User,
  LoginDTO,
  // ... otros tipos según el examen
} from '../types';

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:1495/api';

// Helper: cabeceras con token si existe
function getHeaders(): HeadersInit {
  const headers: HeadersInit = {
    'Content-Type': 'application/json',
  };
  const token = localStorage.getItem('token');
  if (token) {
    headers.Authorization = `Bearer ${token}`;
  }
  return headers;
}

// Helper: parseo de respuesta + error
async function handleResponse<T>(response: Response): Promise<T> {
  if (!response.ok) {
    const error = await response.json().catch(() => ({
      message: `Error ${response.status}`,
    }));
    throw new Error(error.message || `Error ${response.status}`);
  }
  return response.json();
}
```

## AUTH (idéntico en ambos exámenes)

```ts
export const authAPI = {
  async login(data: LoginDTO): Promise<AuthResponse> {
    const response = await fetch(`${API_URL}/auth/login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    return handleResponse<AuthResponse>(response);
  },

  async getMe(): Promise<{ user: User }> {
    const response = await fetch(`${API_URL}/auth/me`, {
      headers: getHeaders(),
    });
    return handleResponse<{ user: User }>(response);
  },
};
```

> **Importante**: `getMe()` **no** recibe el token como parámetro. Lo lee de localStorage a través de `getHeaders()`. Si el enunciado pide explícitamente que reciba el token (Examen 1 lo pide), añádelo:
>
> ```ts
> async getMe(token: string): Promise<{ user: User }> {
>   const response = await fetch(`${API_URL}/auth/me`, {
>     headers: { Authorization: `Bearer ${token}` },
>   });
>   return handleResponse(response);
> }
> ```

## CRUD de un recurso (ejemplo: Incidents)

```ts
import type {
  Incident,
  IncidentsResponse,
  IncidentResponse,
  MessageResponse,
  CreateIncidentDTO,
} from '../types';

export const incidentsAPI = {
  // Listado público
  async getIncidents(): Promise<IncidentsResponse> {
    const response = await fetch(`${API_URL}/incidents`);
    return handleResponse<IncidentsResponse>(response);
  },

  // Crear (requiere token)
  async createIncident(data: CreateIncidentDTO): Promise<IncidentResponse> {
    const response = await fetch(`${API_URL}/incidents`, {
      method: 'POST',
      headers: getHeaders(),
      body: JSON.stringify(data),
    });
    return handleResponse<IncidentResponse>(response);
  },

  // Eliminar (requiere token)
  async deleteIncident(id: number): Promise<MessageResponse> {
    const response = await fetch(`${API_URL}/incidents/${id}`, {
      method: 'DELETE',
      headers: getHeaders(),
    });
    return handleResponse<MessageResponse>(response);
  },
};
```

## Variante para Examen 1 (firmas exactas que pide el enunciado)

El enunciado del Examen 1 pide que las funciones reciban `token` explícito:

```ts
export async function login(credentials: LoginDTO): Promise<AuthResponse> { /* ... */ }
export async function getMe(token: string): Promise<User> { /* ... */ }
export async function getIncidents(): Promise<IncidentsResponse> { /* ... */ }
export async function createIncident(token: string, incident: CreateIncidentDTO): Promise<IncidentResponse> { /* ... */ }
export async function deleteIncident(token: string, id: number): Promise<MessageResponse> { /* ... */ }
```

> **Lee el enunciado**: si pide funciones sueltas (no objeto `xxxAPI`), exporta funciones sueltas. Si pide objeto, usa objeto. **No discutas con el enunciado.**

## Variante para Examen 2 (con paginación y búsqueda)

```ts
export const gamesAPI = {
  async getGames(page = 1, limit = 12, search = ''): Promise<GamesResponse> {
    const params = new URLSearchParams();
    params.set('page', String(page));
    params.set('limit', String(limit));
    if (search) params.set('search', search);

    const response = await fetch(`${API_URL}/games?${params.toString()}`);
    return handleResponse<GamesResponse>(response);
  },

  async getGame(id: number): Promise<GameDetail> {
    const response = await fetch(`${API_URL}/games/${id}`);
    return handleResponse<GameDetail>(response);
  },

  async deleteGame(id: number): Promise<MessageResponse> {
    const response = await fetch(`${API_URL}/games/${id}`, {
      method: 'DELETE',
      headers: getHeaders(),
    });
    return handleResponse<MessageResponse>(response);
  },
};

export const favoritesAPI = {
  async getFavorites(): Promise<Game[]> {
    const response = await fetch(`${API_URL}/favorites`, {
      headers: getHeaders(),
    });
    return handleResponse<Game[]>(response);
  },

  async addFavorite(gameId: number): Promise<MessageResponse> {
    const response = await fetch(`${API_URL}/favorites/${gameId}`, {
      method: 'POST',
      headers: getHeaders(),
    });
    return handleResponse<MessageResponse>(response);
  },

  async removeFavorite(gameId: number): Promise<MessageResponse> {
    const response = await fetch(`${API_URL}/favorites/${gameId}`, {
      method: 'DELETE',
      headers: getHeaders(),
    });
    return handleResponse<MessageResponse>(response);
  },
};

export const aiAPI = {
  async getRecommendations(title: string): Promise<{ recommendations: string }> {
    const response = await fetch(`${API_URL}/ai/recommend`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ title }),
    });
    return handleResponse<{ recommendations: string }>(response);
  },
};
```

## Errores típicos (NO los hagas)

| ❌ Mal | ✅ Bien |
|---|---|
| `fetch(API_URL + '/auth/login')` con concatenación | Template literal: `` `${API_URL}/auth/login` `` |
| Olvidar `Content-Type: application/json` en POST | Siempre incluido en `getHeaders()` |
| Devolver `response` en vez de `response.json()` | `return handleResponse(response)` |
| `Authorization: token` (sin `Bearer `) | `` Authorization: `Bearer ${token}` `` |
| `localStorage.getItem(token)` (variable) | `localStorage.getItem('token')` (literal) |
| No tipar el retorno (`Promise<any>`) | Tipa siempre con los tipos de `types/index.ts` |
