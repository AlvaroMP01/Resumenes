# 06 — LoginPage con `useActionState` (React 19)

**OBLIGATORIO en Examen 2**. El enunciado es explícito: si usas `onSubmit + useState` para el login, ese apartado se considera **incompleto** (puntúa 0%).

## Cómo funciona `useActionState`

```tsx
const [state, formAction, isPending] = useActionState(actionFn, initialState);
```

- `actionFn`: función `async (prevState, formData) => nuevoState`. La llama React automáticamente al hacer submit.
- `initialState`: el estado inicial (objeto con campos que vayas a mostrar).
- `state`: el estado actual (lo que devolvió la última invocación de `actionFn`).
- `formAction`: lo pasas al atributo `action` del `<form>`.
- `isPending`: `true` mientras la action está corriendo.

## Plantilla canónica para el login

```tsx
// src/pages/LoginPage.tsx
import { useActionState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

// Estado del formulario
interface LoginFormState {
  error: string | null;
  success: boolean;
}

const initialState: LoginFormState = {
  error: null,
  success: false,
};

export default function LoginPage() {
  const { login, user } = useAuth();
  const navigate = useNavigate();

  // Action: lo que se ejecuta al enviar el formulario
  async function loginAction(
    _prevState: LoginFormState,
    formData: FormData
  ): Promise<LoginFormState> {
    const email = formData.get('email') as string;
    const password = formData.get('password') as string;

    // Validación básica
    if (!email || !password) {
      return { error: 'Email y contraseña son obligatorios', success: false };
    }

    try {
      await login(email, password);
      return { error: null, success: true };
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Credenciales incorrectas';
      return { error: message, success: false };
    }
  }

  const [state, formAction, isPending] = useActionState(loginAction, initialState);

  // Si el login fue correcto, redirigir
  useEffect(() => {
    if (state.success && user) {
      navigate('/games');
    }
  }, [state.success, user, navigate]);

  return (
    <div className="login-page">
      <h1>Iniciar sesión</h1>

      <form action={formAction} className="login-form">
        <div className="form-group">
          <label htmlFor="email">Email</label>
          <input
            id="email"
            name="email"
            type="email"
            defaultValue="alumno@test.com"
            required
            disabled={isPending}
          />
        </div>

        <div className="form-group">
          <label htmlFor="password">Contraseña</label>
          <input
            id="password"
            name="password"
            type="password"
            defaultValue="123456"
            required
            disabled={isPending}
          />
        </div>

        {state.error && <p className="form-error">{state.error}</p>}

        <button type="submit" disabled={isPending} className="btn btn-primary">
          {isPending ? 'Entrando…' : 'Iniciar sesión'}
        </button>
      </form>

      <p className="login-hint">
        Prueba: <code>alumno@test.com</code> / <code>123456</code>
      </p>
    </div>
  );
}
```

## Diferencias clave vs `useState + onSubmit`

| Aspecto | Clásico (`useState`) | `useActionState` |
|---|---|---|
| Estado de los campos | `useState` por cada campo | `defaultValue` en cada input + `formData.get(...)` |
| Manejador del submit | `onSubmit` con `e.preventDefault()` | `action={formAction}` (sin `onSubmit`) |
| Estado del envío | `useState<boolean>(false)` | `isPending` automático |
| Errores | `useState` o `error` del context | Devolver `{ error, ... }` desde la action |
| Redirección | `navigate(...)` después del `await` | `useEffect` mirando `state.success` |
| Tipos | `FormEvent<HTMLFormElement>` | `(prev, formData: FormData) => Promise<State>` |

## Cosas importantes que NO se ven a simple vista

1. **Los inputs llevan `name`** — sin `name`, `formData.get(name)` devuelve `null`.
2. **`defaultValue` en lugar de `value`** — los inputs son **no controlados** en este patrón. Si pones `value`, React te avisa de que también necesita `onChange`.
3. **`<form action={formAction}>`** — el `action` no es un string, es la función. **NO** uses `onSubmit`.
4. **El `_prevState` casi nunca se usa** — el guion bajo indica eso, está bien.
5. **El `useEffect` para redirigir** mira `state.success`, NO el resultado del `login`. Es la única forma limpia de redirigir tras la action.

## Variante para un formulario CRUD (no login)

Mismo patrón sirve para crear un Game, Incident, Favorite, etc.

```tsx
import { useActionState } from 'react';
import { useGames } from '../context/GamesContext';

interface FormState {
  error: string | null;
  success: string | null;
}

export default function GameForm() {
  const { addGame } = useGames();

  async function action(_prev: FormState, formData: FormData): Promise<FormState> {
    const title = formData.get('title') as string;

    if (!title?.trim()) {
      return { error: 'El título es obligatorio', success: null };
    }

    try {
      await addGame({ title: title.trim() });
      return { error: null, success: 'Juego creado correctamente' };
    } catch (err) {
      return {
        error: err instanceof Error ? err.message : 'Error al crear juego',
        success: null,
      };
    }
  }

  const [state, formAction, isPending] = useActionState(action, {
    error: null,
    success: null,
  });

  return (
    <form action={formAction}>
      <input name="title" disabled={isPending} required />

      {state.error && <p className="error">{state.error}</p>}
      {state.success && <p className="success">{state.success}</p>}

      <button type="submit" disabled={isPending}>
        {isPending ? 'Creando…' : 'Crear'}
      </button>
    </form>
  );
}
```

## Errores frecuentes con `useActionState`

| ❌ Mal | ✅ Bien |
|---|---|
| Inputs sin `name` | Cada input necesita `name="campo"` para `formData.get('campo')` |
| Usar `value` y `onChange` | Usa `defaultValue` (componente no controlado) |
| `<form onSubmit={...}>` | `<form action={formAction}>` (sin onSubmit) |
| Redirigir dentro de la action | Devuelve `{ success: true }` y redirige en `useEffect` |
| No tipar el state | Define `interface FormState` y úsalo como genérico |
| Desestructurar mal: `const [state, action]` | Son TRES: `[state, formAction, isPending]` |
| Ignorar `isPending` | Úsalo para `disabled` en inputs y botón |
