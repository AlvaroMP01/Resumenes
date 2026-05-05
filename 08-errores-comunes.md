# 08 — Errores comunes (checklist anti-suspenso)

Estos errores ya están en el código de `Examen-react/src/` y en `prueba/src/`. **No los repitas.**

## 1. Errores de `localStorage`

```ts
// ❌ Mal: 'token' es la CLAVE, no una variable
localStorage.getItem(token);
localStorage.setItem(response.data.token);  // falta la clave
localStorage.removeItem(token);

// ✅ Bien
localStorage.getItem('token');
localStorage.setItem('token', response.token);
localStorage.removeItem('token');
```

## 2. Recursión infinita por mismo nombre

```ts
// ❌ Mal: dentro de la función login, llamas a 'login' que es ESTA función
async function login(data: LoginDTO) {
  const response = await login(data); // ← se llama a sí misma → stack overflow
}

// ✅ Bien: importas la del service y le pones nombre diferente
import { authAPI } from '../services/api';

async function login(data: LoginDTO) {
  const response = await authAPI.login(data);
}
```

## 3. Tipos mal usados (`AuthResponse`)

Mira tu `types/index.ts` y comprueba la **forma exacta** de `AuthResponse`:

```ts
// Examen 1: respuesta envuelta
interface AuthResponse {
  ok: boolean;
  data?: { user: User; token: string };
  error?: string;
}
// → response.data?.user, response.data?.token

// Examen 2: respuesta directa
interface AuthResponse {
  user: User;
  token: string;
}
// → response.user, response.token
```

```ts
// ❌ Mal en Examen 1
setUser(response.user);          // undefined
setToken(response.token);        // undefined

// ✅ Bien en Examen 1
setUser(response.data?.user ?? null);
setToken(response.data?.token ?? null);
```

## 4. `useEffect` sin dependencias correctas

```ts
// ❌ Mal: se ejecuta en cada render
useEffect(() => {
  fetchIncidents();
}); // sin array → infinitas peticiones

// ✅ Bien: solo al montar
useEffect(() => {
  fetchIncidents();
}, []);

// ✅ Bien: cuando cambia algo externo
useEffect(() => {
  fetchFavorites();
}, [token]);
```

## 5. Mutar el estado en lugar de crear array nuevo

```ts
// ❌ Mal: muta el array original (React no detecta el cambio)
items.push(nuevo);
setItems(items);

items.splice(index, 1);
setItems(items);

// ✅ Bien: array nuevo (referencia distinta → React re-renderiza)
setItems((prev) => [...prev, nuevo]);
setItems((prev) => prev.filter((x) => x.id !== id));
setItems((prev) => prev.map((x) => (x.id === id ? actualizado : x)));
```

## 6. Olvidar `e.preventDefault()` en formularios clásicos

```tsx
// ❌ Mal: la página se recarga al enviar
<form onSubmit={(e) => login(...)}>

// ✅ Bien
<form onSubmit={(e) => {
  e.preventDefault();
  login(...);
}}>
```

> En `useActionState` esto NO aplica — el `<form action={formAction}>` no recarga.

## 7. `ProtectedRoute` que redirige sin esperar `loading`

```tsx
// ❌ Mal: si checkAuth aún corre, user es null y redirige
if (!user) return <Navigate to="/login" replace />;

// ✅ Bien: primero esperamos a que termine la comprobación
if (loading) return <p>Comprobando sesión…</p>;
if (!user) return <Navigate to="/login" replace />;
return <>{children}</>;
```

## 8. `BrowserRouter` mal colocado

```tsx
// ❌ Mal: el Navbar usa Link y rompe (no hay Router en su contexto)
<BrowserRouter>
  <AuthProvider>
    <Navbar />  {/* ← este Navbar está fuera del Router */}
    <Routes>...</Routes>
  </AuthProvider>
</BrowserRouter>

// ✅ Bien: providers fuera, router dentro
<AuthProvider>
  <BrowserRouter>
    <Navbar />
    <Routes>...</Routes>
  </BrowserRouter>
</AuthProvider>
```

## 9. Inputs no controlados pero usando `value` (warning de React)

```tsx
// ❌ Mal: warning "A component is changing an uncontrolled input to controlled"
<input value={title} />              // value sin onChange
<input onChange={(e) => setTitle(e.target.value)} />  // onChange sin value

// ✅ Bien: ambos juntos (controlado)
<input value={title} onChange={(e) => setTitle(e.target.value)} />

// ✅ Bien: ninguno (no controlado, leer con FormData o ref)
<input defaultValue={title} />
```

## 10. `useActionState` con `value` en lugar de `defaultValue`

```tsx
// ❌ Mal: React avisa de que necesitas onChange
<input name="email" value="alumno@test.com" />

// ✅ Bien: en useActionState los inputs son no controlados
<input name="email" defaultValue="alumno@test.com" />
```

## 11. Olvidar el `name` en inputs con `useActionState`

```tsx
// ❌ Mal: formData.get('email') devuelve null
<input id="email" type="email" />

// ✅ Bien
<input id="email" name="email" type="email" />
```

## 12. Importar de `react-router` en lugar de `react-router-dom`

```ts
// ❌ Mal
import { BrowserRouter, Routes, Route } from 'react-router';

// ✅ Bien
import { BrowserRouter, Routes, Route } from 'react-router-dom';
```

## 13. Crear tipos nuevos en lugar de importar los que ya están

```ts
// ❌ Mal: redefines el tipo en el contexto
interface Incident {
  id: number;
  title: string;
  // ...
}

// ✅ Bien: importas del fichero proporcionado
import type { Incident } from '../types';
```

## 14. Usar `Toaster` / `sonner` cuando NO está instalado

El profe a veces lo usa en sus ejemplos, pero **NO viene en el package.json del examen**:

```ts
// ❌ Mal: import { toast } from 'sonner'  → error "Cannot find module"
toast.success('Login exitoso');

// ✅ Bien: usa estado local para mensajes
const [success, setSuccess] = useState<string | null>(null);
// ...
setSuccess('Login exitoso');
{success && <p className="success">{success}</p>}
```

## 15. Olvidar limpiar el formulario tras crear

El enunciado del Examen 1 lo exige explícitamente:

```tsx
// ❌ Mal: tras crear, los campos siguen rellenos
await addIncident(data);

// ✅ Bien: reseteas todos los campos
await addIncident(data);
setTitle('');
setDescription('');
setPriority('media');
```

## 16. Concat en URL en lugar de URLSearchParams

```ts
// ❌ Mal: si el search tiene espacios, se rompe
fetch(`${API_URL}/games?search=${search}&limit=${limit}`);

// ✅ Bien
const params = new URLSearchParams();
if (search) params.set('search', search);
params.set('limit', String(limit));
fetch(`${API_URL}/games?${params.toString()}`);
```

## 17. Olvidar el `key` en `.map`

```tsx
// ❌ Mal: warning rojo en consola, puntúa el apartado de calidad
{items.map((item) => <ItemCard item={item} />)}

// ✅ Bien
{items.map((item) => <ItemCard key={item.id} item={item} />)}
```

## 18. Errores de TypeScript que dejas en el código

El Examen 2, apartado 8 ("Calidad y robustez"), pide expresamente **0 errores de TypeScript**. Antes de entregar:

```bash
# Comprueba que TS está limpio
npm run build
```

Si build falla, **arregla los errores antes de entregar**. Aunque el profe dice "No build después de cambios" en otros contextos, en el examen sí conviene comprobar.

## 19. Subir `node_modules` en el ZIP

```bash
# ❌ Mal: 200 MB de basura
zip -r examen-juan-gp.zip examen-juan-gp/

# ✅ Bien: excluye node_modules
zip -r examen-juan-gp.zip examen-juan-gp/ -x "*/node_modules/*"
```

O directamente borra `node_modules/` antes de comprimir.

---

## Checklist final antes de subir el ZIP

- [ ] El backend responde a `http://IP_BACKEND/api/incidents` (o el endpoint que sea).
- [ ] `npm run dev` arranca sin errores rojos en consola.
- [ ] Login con `alumno@test.com / 123456` funciona.
- [ ] Tras login, recargar la página → sigues logueado.
- [ ] Crear un item → aparece en la lista al instante.
- [ ] Eliminar un item → desaparece al instante.
- [ ] Logout → vuelves al estado público, token desaparece de localStorage.
- [ ] Sin `node_modules/` en la carpeta.
- [ ] Carpeta renombrada según formato pedido.
- [ ] Sin warnings rojos de React (sin keys, controlled/uncontrolled, etc.).
- [ ] Sin `console.log` de depuración por todos lados (deja como mucho 1–2 si tiene sentido).
- [ ] El Footer (Examen 2) tiene tu nombre real y tu GitHub real.
