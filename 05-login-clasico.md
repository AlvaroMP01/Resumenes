# 05 — LoginPage clásico (`useState` + `onSubmit`)

Formato del **Examen 1 (IncidTech)**. NO uses esto en el Examen 2 (allí es obligatorio `useActionState`).

## Reglas

- Dos `useState` para `email` y `password` (campos controlados).
- Un `onSubmit` que hace `e.preventDefault()` y llama al `login()` del contexto.
- Si hay `error` del contexto, mostrarlo en pantalla.
- Si el login es correcto, redirigir a `/incidents` con `useNavigate`.

## Plantilla canónica

```tsx
// src/pages/LoginPage.tsx
import { useState, type FormEvent } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '../context/AuthContext';

export default function LoginPage() {
  const { login, error } = useAuth();
  const navigate = useNavigate();

  const [email, setEmail] = useState<string>('');
  const [password, setPassword] = useState<string>('');
  const [submitting, setSubmitting] = useState<boolean>(false);

  async function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();

    // Validación básica
    if (!email || !password) return;

    try {
      setSubmitting(true);
      await login(email, password);
      navigate('/incidents'); // o donde diga el enunciado
    } catch {
      // El error ya queda guardado en `error` del AuthContext
    } finally {
      setSubmitting(false);
    }
  }

  return (
    <div className="login-page">
      <h1>Iniciar sesión</h1>

      <form onSubmit={handleSubmit} className="login-form">
        <div className="form-group">
          <label htmlFor="email">Email</label>
          <input
            id="email"
            type="email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
            disabled={submitting}
          />
        </div>

        <div className="form-group">
          <label htmlFor="password">Contraseña</label>
          <input
            id="password"
            type="password"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            required
            disabled={submitting}
          />
        </div>

        {/* Mostrar error si lo hay */}
        {error && <p className="form-error">{error}</p>}

        <button type="submit" disabled={submitting} className="btn btn-primary">
          {submitting ? 'Entrando…' : 'Iniciar sesión'}
        </button>
      </form>

      <p className="login-hint">
        Usa <code>alumno@test.com</code> / <code>123456</code> para probar.
      </p>
    </div>
  );
}
```

## Formulario para crear un recurso (`IncidentForm`)

Mismo patrón, sirve para Apartado 4.2 del Examen 1.

```tsx
// src/components/IncidentForm.tsx
import { useState, type FormEvent } from 'react';
import { useNavigate } from 'react-router-dom';
import { useIncidents } from '../context/IncidentsContext';
import type { CreateIncidentDTO } from '../types';

export default function IncidentForm() {
  const { addIncident } = useIncidents();
  const navigate = useNavigate();

  const [title, setTitle] = useState<string>('');
  const [description, setDescription] = useState<string>('');
  const [priority, setPriority] = useState<'alta' | 'media' | 'baja'>('media');
  const [submitting, setSubmitting] = useState<boolean>(false);

  async function handleSubmit(e: FormEvent<HTMLFormElement>) {
    e.preventDefault();

    // Validación básica obligatoria por enunciado
    if (!title.trim() || !description.trim()) return;

    const data: CreateIncidentDTO = {
      title: title.trim(),
      description: description.trim(),
      priority,
    };

    try {
      setSubmitting(true);
      await addIncident(data);

      // Limpiar formulario tras crear (lo pide el enunciado)
      setTitle('');
      setDescription('');
      setPriority('media');

      navigate('/incidents'); // opcional: redirigir a la lista
    } catch (err) {
      console.error('Error al crear incidencia:', err);
    } finally {
      setSubmitting(false);
    }
  }

  return (
    <form onSubmit={handleSubmit} className="incident-form">
      <div className="form-group">
        <label htmlFor="title">Título</label>
        <input
          id="title"
          type="text"
          value={title}
          onChange={(e) => setTitle(e.target.value)}
          placeholder="Describe brevemente la incidencia"
          required
          disabled={submitting}
        />
      </div>

      <div className="form-group">
        <label htmlFor="description">Descripción</label>
        <textarea
          id="description"
          value={description}
          onChange={(e) => setDescription(e.target.value)}
          placeholder="Explica la incidencia con detalle"
          rows={4}
          required
          disabled={submitting}
        />
      </div>

      <div className="form-group">
        <label htmlFor="priority">Prioridad</label>
        <select
          id="priority"
          value={priority}
          onChange={(e) => setPriority(e.target.value as 'alta' | 'media' | 'baja')}
          disabled={submitting}
        >
          <option value="alta">Alta</option>
          <option value="media">Media</option>
          <option value="baja">Baja</option>
        </select>
      </div>

      <button type="submit" disabled={submitting} className="btn btn-primary">
        {submitting ? 'Creando…' : 'Crear incidencia'}
      </button>
    </form>
  );
}
```

## Tarjeta de un item con renderizado condicional (`IncidentCard`)

Apartado 4.1 del Examen 1. Usa `useAuth` para el botón "Eliminar".

```tsx
// src/components/IncidentCard.tsx
import { useAuth } from '../context/AuthContext';
import { useIncidents } from '../context/IncidentsContext';
import type { Incident } from '../types';

interface IncidentCardProps {
  incident: Incident;
}

export default function IncidentCard({ incident }: IncidentCardProps) {
  const { user } = useAuth();
  const { removeIncident } = useIncidents();

  async function handleDelete() {
    if (!confirm('¿Seguro que quieres eliminar esta incidencia?')) return;
    await removeIncident(incident.id);
  }

  return (
    <article className="incident-card">
      <header className="incident-header">
        <h3>{incident.title}</h3>
        {/* Clase dinámica según prioridad */}
        <span className={`badge badge-${incident.priority}`}>
          {incident.priority}
        </span>
        <span className={`badge badge-${incident.status}`}>
          {incident.status}
        </span>
      </header>

      <p className="incident-description">{incident.description}</p>

      <footer className="incident-footer">
        <time>{new Date(incident.createdAt).toLocaleDateString()}</time>

        {/* Botón solo visible si hay sesión */}
        {user && (
          <button onClick={handleDelete} className="btn btn-danger">
            Eliminar
          </button>
        )}
      </footer>
    </article>
  );
}
```

## Página de lista (`IncidentsPage`)

```tsx
// src/pages/IncidentsPage.tsx
import { useIncidents } from '../context/IncidentsContext';
import IncidentCard from '../components/IncidentCard';

export default function IncidentsPage() {
  const { incidents, loading } = useIncidents();

  if (loading) {
    return <p className="loading">Cargando incidencias…</p>;
  }

  if (incidents.length === 0) {
    return <p className="empty">No hay incidencias registradas</p>;
  }

  return (
    <section className="incidents-page">
      <h1>Incidencias</h1>
      <div className="incidents-grid">
        {incidents.map((incident) => (
          <IncidentCard key={incident.id} incident={incident} />
        ))}
      </div>
    </section>
  );
}
```

## Errores frecuentes

| ❌ Mal | ✅ Bien |
|---|---|
| Olvidar `e.preventDefault()` | SIEMPRE en el `onSubmit` |
| `value` sin `onChange` (input incontrolado) | Los dos juntos |
| No deshabilitar el botón con `submitting` | `disabled={submitting}` para evitar doble submit |
| Validar SOLO en el backend | Validación básica en el form: `if (!x.trim()) return` |
| Olvidar limpiar el formulario tras crear | `setTitle(''); setDescription('');` |
| Usar `confirm()` sin `if` | `if (!confirm(...)) return;` antes de borrar |
| Tipar el evento como `any` | `FormEvent<HTMLFormElement>` |
