# 02 — `AuthContext` + `useAuth`

Este es el contexto **más importante** del examen. Si lo haces mal, todo lo que depende de él (Navbar, ProtectedRoute, Login) falla.

## Lo que tiene que exponer

| Campo | Tipo | Cuándo se actualiza |
|---|---|---|
| `user` | `User \| null` | Tras `login` correcto, `getMe` correcto, o `logout` |
| `token` | `string \| null` | Igual que `user` |
| `loading` | `boolean` | `true` al inicio (antes de `checkAuth`); `false` cuando termina |
| `error` | `string \| null` *(opcional)* | Si falla el login con credenciales malas |
| `isAuthenticated` | `boolean` *(opcional)* | `!!user` — útil en otros sitios |
| `login(...)` | `(data) => Promise<...>` | Función para iniciar sesión |
| `logout()` | `() => void` | Limpia user, token y localStorage |

> **Mira tu enunciado**: el Examen 1 pide `error: string | null` en el contexto, pero el Examen 2 NO. Adapta los campos a lo que pida el PDF.

## Plantilla canónica (versión completa con error)

```tsx
// src/context/AuthContext.tsx
import { createContext, useContext, useEffect, useState, type ReactNode } from 'react';
import { authAPI } from '../services/api';
import type { User, LoginDTO } from '../types';

// 1. Tipo del contexto
export interface AuthContextType {
  user: User | null;
  token: string | null;
  loading: boolean;
  error: string | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

// 2. Contexto (undefined al inicio para forzar uso dentro del Provider)
const AuthContext = createContext<AuthContextType | undefined>(undefined);

// 3. Provider
export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  // Lazy init: leer token de localStorage solo la primera vez
  const [token, setToken] = useState<string | null>(() =>
    localStorage.getItem('token')
  );
  const [loading, setLoading] = useState<boolean>(true);
  const [error, setError] = useState<string | null>(null);

  // 4. Comprobación inicial: si hay token guardado, validarlo con /auth/me
  useEffect(() => {
    async function checkAuth() {
      const savedToken = localStorage.getItem('token');
      if (!savedToken) {
        setLoading(false);
        return;
      }
      try {
        // Si el examen pide getMe(token), pásalo: authAPI.getMe(savedToken)
        const response = await authAPI.getMe();
        setUser(response.user);
        setToken(savedToken);
      } catch {
        // Token inválido o expirado → limpiar
        localStorage.removeItem('token');
        setToken(null);
        setUser(null);
      } finally {
        setLoading(false);
      }
    }
    checkAuth();
  }, []);

  // 5. Login
  async function login(email: string, password: string): Promise<void> {
    try {
      setError(null);
      const data: LoginDTO = { email, password };
      const response = await authAPI.login(data);

      // Adapta esto según la forma EXACTA de tu AuthResponse:
      //   - Examen 1: { ok, data: { user, token } }
      //   - Examen 2: { user, token }
      // Mira tu types/index.ts y ajusta:
      const userData = response.data?.user ?? response.user;
      const tokenData = response.data?.token ?? response.token;

      if (!userData || !tokenData) {
        throw new Error('Respuesta de login inválida');
      }

      localStorage.setItem('token', tokenData);
      setToken(tokenData);
      setUser(userData);
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Credenciales incorrectas';
      setError(message);
      throw err; // Re-lanzo para que el LoginPage pueda reaccionar si quiere
    }
  }

  // 6. Logout
  function logout(): void {
    localStorage.removeItem('token');
    setToken(null);
    setUser(null);
  }

  const value: AuthContextType = {
    user,
    token,
    loading,
    error,
    login,
    logout,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

// 7. Custom hook
export function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth debe usarse dentro de AuthProvider');
  }
  return context;
}
```

## Variante minimalista (sin `error`, para Examen 2)

Si el enunciado no pide `error`, simplifica el `login` así:

```tsx
async function login(email: string, password: string): Promise<void> {
  const data: LoginDTO = { email, password };
  const response = await authAPI.login(data);
  localStorage.setItem('token', response.token);
  setToken(response.token);
  setUser(response.user);
}
```

Y en el LoginPage manejas el error con `try/catch` local.

## Variante con `useAuth` en fichero separado

Algunos exámenes piden `hooks/useAuth.ts` aparte. Si es así:

```tsx
// src/hooks/useAuth.ts
import { useContext } from 'react';
import { AuthContext } from '../context/AuthContext';

export function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth debe usarse dentro de AuthProvider');
  }
  return context;
}
```

Y en `AuthContext.tsx` solo exportas `AuthContext` y `AuthProvider`.

## Checklist (verifica TODOS antes de pasar al siguiente apartado)

- [ ] `loading` empieza en `true` y pasa a `false` cuando termina `checkAuth`.
- [ ] Token se guarda como `localStorage.setItem('token', token)`. **Clave entre comillas**.
- [ ] Tras `login` correcto, `user` y `token` quedan poblados.
- [ ] Tras `logout`, `localStorage.removeItem('token')` y todo a `null`.
- [ ] `useAuth()` lanza error si se usa fuera del provider.
- [ ] Si recargas la página estando logueado, sigues logueado (gracias a `checkAuth`).

## Errores frecuentes (vienen ya en `Examen-react/src/contexts/AuthContext.tsx`)

```ts
// ❌ NO hagas esto
const savedToken = localStorage.getItem(token);     // 'token' es la clave, no la variable
const response = await login(data);                  // recursión infinita: te llamas a ti mismo
localStorage.setItem(response.data.token);           // falta la clave, error de TS
const meResponse = await getMe(savedToken);          // función no importada

// ✅ Forma correcta
const savedToken = localStorage.getItem('token');
const response = await authAPI.login(data);          // llama al service
localStorage.setItem('token', response.token);       // clave + valor
const meResponse = await authAPI.getMe();            // o authAPI.getMe(savedToken)
```
