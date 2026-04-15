# Tutorial Lengkap: Membangun Todo App dengan Next.js + Supabase

Tutorial ini akan menjelaskan **dari nol** bagaimana aplikasi Todo ini bekerja — mulai dari struktur folder, cara setiap file saling berkomunikasi, hingga alur data dari frontend ke database dan kembali.

**Tech Stack:**
- **Next.js 16** (App Router) — menangani routing & rendering
- **Supabase** — backend (database PostgreSQL, Auth, Realtime)
- **Shadcn UI** + **Tailwind CSS** — komponen UI
- **Server Actions** — komunikasi frontend ↔ backend tanpa API endpoint terpisah
- **TypeScript** — type safety di seluruh aplikasi

---

## 📂 Struktur Folder Proyek

```
todo-supabase/
├── app/                          ← ROUTES (halaman & backend logic)
│   ├── todos/
│   │   └── page.tsx              ← Halaman utama: GET todos dari DB, render UI
│   ├── actions.ts                ← SERVER ACTIONS: POST/UPDATE/DELETE todos
│   ├── layout.tsx                ← Root layout: HTML wrapper, font, metadata
│   ├── page.tsx                  ← Landing page: redirect ke /todos
│   ├── globals.css               ← Global CSS (Tailwind + Shadcn)
│   └── favicon.ico
├── components/                   ← UI COMPONENTS (reusable)
│   ├── ui/                       ← Komponen dari Shadcn (button, input, card, dll)
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── card.tsx
│   │   ├── checkbox.tsx
│   │   └── ...
│   ├── add-todo-form.tsx         ← Form untuk tambah todo baru
│   ├── todo-list.tsx             ← List todos + realtime subscription
│   └── todo-item.tsx             ← Satu item todo (checkbox + delete)
├── lib/                          ← UTILITIES & CLIENTS
│   └── supabase/
│       ├── server.ts             ← Supabase client untuk SERVER (Server Components/Actions)
│       └── client.ts             ← Supabase client untuk BROWSER (Client Components)
├── types/
│   └── database.ts               ← TypeScript types dari schema Supabase (auto-generated)
├── .env.local                    ← Environment variables (URL & API key Supabase)
├── components.json               ← Konfigurasi Shadcn UI
├── package.json                  ← Dependencies & scripts
├── next.config.ts                ← Konfigurasi Next.js
├── tsconfig.json                 ← Konfigurasi TypeScript
└── tailwind.config.ts            ← Konfigurasi Tailwind CSS
```

---

## 🧠 Konsep Dasar yang Wajib Dipahami

### 1. Server Components vs Client Components

| | Server Components | Client Components |
|---|---|---|
| **Jalan di mana?** | Server Next.js | Browser user |
| **Bisa akses DB?** | ✅ Langsung query DB | ❌ Harus lewat API/Server Action |
| **Bisa pakai useState/useEffect?** | ❌ | ✅ |
| **Directive** | Default (tanpa `'use client'`) | Harus ada `'use client'` di atas file |
| **Contoh di proyek ini** | `app/todos/page.tsx`, `app/actions.ts` | `components/add-todo-form.tsx`, `components/todo-list.tsx` |

### 2. File-based Routing (App Router)

Di Next.js App Router, **struktur folder = URL**:

```
app/page.tsx          → http://localhost:3000/
app/todos/page.tsx    → http://localhost:3000/todos
```

File bernama `page.tsx` di dalam folder akan jadi halaman untuk route tersebut.

### 3. Server Actions

Server Actions adalah fungsi yang:
- Ditandai dengan `'use server'` di atas file
- **Jalan di server**, bukan di browser
- Bisa dipanggil langsung dari komponen client (form action, onClick, dll)
- Tidak perlu bikin API route terpisah

Di proyek ini, semua operasi database (add, toggle, delete todo) dilakukan lewat Server Actions di `app/actions.ts`.

---

## 🔄 Alur Komunikasi: Frontend ↔ Backend ↔ Database

Berikut adalah diagram alur saat user **menambahkan todo baru**:

```
┌─────────────────────────────────────────────────────────────────┐
│                        BROWSER (Client)                         │
│                                                                 │
│  1. User ketik di input → klik "Add Todo"                       │
│     (components/add-todo-form.tsx)                              │
│                                                                 │
│  2. Form submit → memanggil Server Action `addTodo(formData)`   │
│     → HTTP POST otomatis dikirim ke server Next.js              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SERVER NEXT.JS (Backend)                      │
│                                                                 │
│  3. Fungsi `addTodo` di `app/actions.ts` dieksekusi di server   │
│                                                                 │
│  4. `createClient()` dari `lib/supabase/server.ts`              │
│     → Membuat Supabase client (dengan cookie session)           │
│                                                                 │
│  5. `supabase.from('todos').insert({ title })`                  │
│     → Mengirim query INSERT ke Supabase API                     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                     SUPABASE (Database)                          │
│                                                                 │
│  6. Supabase menerima request → check RLS policies              │
│     (kita sudah DISABLE RLS, jadi semua request diperbolehkan)  │
│                                                                 │
│  7. Data INSERT ke tabel `todos` di PostgreSQL                  │
│                                                                 │
│  8. Supabase mengirim event REALTIME via WebSocket              │
│     → ke semua client yang subscribe (di TodoList)              │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                  KEMBALI KE BROWSER (Client)                     │
│                                                                 │
│  9a. `revalidatePath('/todos')` di Server Action                │
│     → Next.js refresh cache → halaman /todos update             │
│                                                                 │
│  9b. Realtime event diterima di `components/todo-list.tsx`      │
│     → `setTodos()` update state → React re-render UI            │
│     → Todo baru muncul di layar TANPA refresh halaman!          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 📖 Penjelasan File per File

### 1. `app/layout.tsx` — Root Layout

**Apa ini?** Wrapper HTML yang membungkus **semua halaman** di aplikasi.

```tsx
import './globals.css'
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
  title: 'Todo App',
  description: 'Full-stack Next.js + Supabase + Shadcn',
}

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body className={inter.className}>{children}</body>
    </html>
  )
}
```

**Yang terjadi:**
- Memuat font `Inter` dari Google Fonts
- Menambahkan global CSS dari `globals.css` (yang sudah berisi Tailwind directives)
- Mengatur `<title>` dan `<meta description>` untuk SEO
- `<body>` berisi `{children}` → ini akan diisi konten dari setiap halaman (`page.tsx`)

**Server Component** (default, tidak ada `'use client'`).

---

### 2. `app/page.tsx` — Landing Page (Root `/`)

**Apa ini?** Halaman yang muncul saat buka `http://localhost:3000`.

```tsx
import { redirect } from 'next/navigation'

export default function Home() {
  redirect('/todos')
}
```

**Yang terjadi:**
- Sangat sederhana: langsung redirect ke `/todos`
- Tidak ada login/signup lagi — semua langsung bisa akses todo list

**Server Component** (karena `redirect()` hanya bisa di server).

---

### 3. `app/todos/page.tsx` — Halaman Todo List

**Apa ini?** Halaman utama aplikasi — mengambil data dari database dan render UI.

```tsx
import { createClient } from '@/lib/supabase/server'
import { TodoList } from '@/components/todo-list'
import { AddTodoForm } from '@/components/add-todo-form'

export default async function TodosPage() {
  const supabase = await createClient()

  const { data: todos } = await supabase
    .from('todos')
    .select('*')
    .order('created_at', { ascending: false })

  return (
    <main className="container mx-auto max-w-2xl py-8 px-4">
      <div className="flex justify-between items-center mb-8">
        <h1 className="text-3xl font-bold">My Todos</h1>
      </div>
      <div className="mb-8">
        <AddTodoForm />
      </div>
      <TodoList initialTodos={todos || []} />
    </main>
  )
}
```

**Yang terjadi:**
1. `async function` → ini **async Server Component** (Next.js 15+ support)
2. `createClient()` → bikin Supabase client versi **server** (dari `lib/supabase/server.ts`)
3. `supabase.from('todos').select('*')` → query **SELECT** ke database, ambil semua todos
4. `.order('created_at', { ascending: false })` → urutkan dari yang terbaru
5. Data `todos` dikirim sebagai **props** ke komponen `<TodoList initialTodos={todos} />`
6. Render `<AddTodoForm />` untuk form tambah todo

**Hubungan dengan file lain:**
- Import `createClient` dari `lib/supabase/server.ts` → untuk akses database di server
- Import `<TodoList>` dari `components/todo-list.tsx` → untuk render list todos
- Import `<AddTodoForm>` dari `components/add-todo-form.tsx` → untuk form tambah todo

**Server Component** (async, query database langsung di server).

---

### 4. `app/actions.ts` — Server Actions (Backend Logic)

**Apa ini?** File yang berisi semua fungsi backend untuk operasi database. Ini adalah "API" dari aplikasi kita.

```ts
'use server'

import { revalidatePath } from 'next/cache'
import { createClient } from '@/lib/supabase/server'

export async function addTodo(formData: FormData) {
  const supabase = await createClient()
  const title = formData.get('title') as string

  const { error } = await supabase
    .from('todos')
    .insert({ title })

  if (error) throw new Error(`Failed to add todo: ${error.message} (code: ${error.code})`)
  revalidatePath('/todos')
}

export async function toggleTodo(id: string, isComplete: boolean) {
  const supabase = await createClient()
  const { error } = await supabase
    .from('todos')
    .update({ is_complete: !isComplete, updated_at: new Date().toISOString() })
    .eq('id', id)

  if (error) throw new Error('Failed to update todo')
  revalidatePath('/todos')
}

export async function deleteTodo(id: string) {
  const supabase = await createClient()
  const { error } = await supabase.from('todos').delete().eq('id', id)
  if (error) throw new Error('Failed to delete todo')
  revalidatePath('/todos')
}
```

**Yang terjadi:**

**`'use server'`** → directive yang bilang Next.js: "fungsi-fungsi di file ini jalan di server, bukan di browser".

**`addTodo(formData)`**:
- Menerima `FormData` dari form di browser
- `supabase.from('todos').insert({ title })` → INSERT ke database
- `revalidatePath('/todos')` → bilang Next.js: "refresh cache halaman /todos" → data baru muncul

**`toggleTodo(id, isComplete)`**:
- UPDATE status `is_complete` di database
- `revalidatePath('/todos')` → refresh cache

**`deleteTodo(id)`**:
- DELETE dari database
- `revalidatePath('/todos')` → refresh cache

**Hubungan dengan file lain:**
- Dipanggil dari **Client Components** (`add-todo-form.tsx`, `todo-item.tsx`)
- Menggunakan `createClient` dari `lib/supabase/server.ts` untuk akses database
- `revalidatePath` dari `next/cache` → memberitahu Next.js untuk refresh halaman

**Server Action** (wajib `'use server'`).

---

### 5. `lib/supabase/server.ts` — Supabase Client untuk Server

**Apa ini?** Fungsi pembantu untuk membuat Supabase client yang bisa jalan di **server**.

```ts
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export const createClient = async () => {
  const cookieStore = await cookies()
  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value
        },
        async set(name: string, value: string, options: any) {
          try {
            cookieStore.set({ name, value, ...options })
          } catch {
            // Set cookie di Server Component (bisa gagal di render time)
          }
        },
        async remove(name: string, options: any) {
          try {
            cookieStore.set({ name, value: '', ...options })
          } catch {
            // Remove cookie di Server Component
          }
        },
      },
    }
  )
}
```

**Yang terjadi:**
- `createServerClient` dari `@supabase/ssr` → versi Supabase client untuk server-side
- `cookies()` dari `next/headers` → akses cookies browser (untuk session/auth)
- **Next.js 15+** → `cookies()` sekarang **async**, makanya `await cookies()`
- Cookie handling → Supabase perlu baca/tulis cookie untuk maintain session user

**Kenapa ada `try-catch` di `set` dan `remove`?**
Karena di Server Components (bukan Server Actions), kita tidak bisa set cookie — cookie hanya bisa di-set di Server Actions atau Route Handlers. `try-catch` mencegah error crash.

**Dipanggil dari:**
- `app/todos/page.tsx` → untuk query database di server
- `app/actions.ts` → untuk operasi database di Server Actions

---

### 6. `lib/supabase/client.ts` — Supabase Client untuk Browser

**Apa ini?** Fungsi pembantu untuk membuat Supabase client yang bisa jalan di **browser**.

```ts
import { createBrowserClient } from '@supabase/ssr'

export const createClient = () =>
  createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
```

**Yang terjadi:**
- `createBrowserClient` → versi Supabase client untuk browser
- Menggunakan environment variables yang sama (`.env.local`)
- **Tidak async** → karena di browser tidak perlu akses cookies Next.js

**Perbedaan dengan `server.ts`:**

| | `server.ts` | `client.ts` |
|---|---|---|
| **Pakai** | `createServerClient` | `createBrowserClient` |
| **Async?** | ✅ `async () =>` | ❌ `() =>` |
| **Cookie handling?** | ✅ Baca/tulis cookies | ❌ Tidak perlu |
| **Dipakai di** | Server Components, Server Actions | Client Components (`'use client'`) |

**Dipanggil dari:**
- `components/todo-list.tsx` → untuk realtime subscription di browser

---

### 7. `components/add-todo-form.tsx` — Form Tambah Todo

**Apa ini?** Form input untuk menambah todo baru. Ini **Client Component**.

```tsx
'use client'

import { useRef } from 'react'
import { useFormStatus } from 'react-dom'
import { addTodo } from '@/app/actions'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Card, CardContent } from '@/components/ui/card'

function SubmitButton() {
  const { pending } = useFormStatus()
  return (
    <Button type="submit" disabled={pending}>
      {pending ? 'Adding...' : 'Add Todo'}
    </Button>
  )
}

export function AddTodoForm() {
  const formRef = useRef<HTMLFormElement>(null)

  return (
    <Card>
      <CardContent className="pt-6">
        <form
          ref={formRef}
          action={async (formData: FormData) => {
            await addTodo(formData)
            formRef.current?.reset()
          }}
          className="flex gap-2"
        >
          <Input
            type="text"
            name="title"
            placeholder="What needs to be done?"
            required
            className="flex-1"
          />
          <SubmitButton />
        </form>
      </CardContent>
    </Card>
  )
}
```

**Yang terjadi:**

**`'use client'`** → directive: komponen ini jalan di browser, bukan di server.

**`useFormStatus`** dari `react-dom` → hook yang kasih tau status form (apakah sedang submit/pending). Ini dipakai di `SubmitButton` untuk tampilkan "Adding..." saat submit.

**`useRef`** dari `react` → referensi ke DOM element form. Dipakai untuk `formRef.current?.reset()` setelah submit berhasil.

**`action={async (formData) => { ... }}`** → ini cara memanggil Server Action dari form:
1. User isi input → klik "Add Todo"
2. Form submit → `formData` berisi `{ title: "isi input" }`
3. `addTodo(formData)` dipanggil → ini **Server Action** dari `app/actions.ts`
4. Server memproses → INSERT ke database
5. Setelah selesai → `formRef.current?.reset()` → input jadi kosong

**Hubungan dengan file lain:**
- Import `addTodo` dari `app/actions.ts` → Server Action untuk tambah todo
- Import komponen UI dari `components/ui/` → Button, Input, Card (dari Shadcn)

---

### 8. `components/todo-list.tsx` — List Todos + Realtime

**Apa ini?** Komponen yang menampilkan daftar todos dan **mendengarkan perubahan realtime** dari Supabase.

```tsx
'use client'

import { useEffect, useState } from 'react'
import { createClient } from '@/lib/supabase/client'
import { TodoItem } from './todo-item'
import type { Database } from '@/types/database'

type Todo = Database['public']['Tables']['todos']['Row']

interface TodoListProps {
  initialTodos: Todo[]
}

export function TodoList({ initialTodos }: TodoListProps) {
  const [todos, setTodos] = useState<Todo[]>(initialTodos)
  const supabase = createClient()

  useEffect(() => {
    const channel = supabase
      .channel('realtime todos')
      .on(
        'postgres_changes',
        { event: '*', schema: 'public', table: 'todos' },
        (payload) => {
          if (payload.eventType === 'INSERT') {
            setTodos((prev) => [payload.new as Todo, ...prev])
          } else if (payload.eventType === 'UPDATE') {
            setTodos((prev) =>
              prev.map((todo) =>
                todo.id === payload.new.id ? (payload.new as Todo) : todo
              )
            )
          } else if (payload.eventType === 'DELETE') {
            setTodos((prev) => prev.filter((todo) => todo.id !== payload.old.id))
          }
        }
      )
      .subscribe()

    return () => {
      supabase.removeChannel(channel)
    }
  }, [supabase])

  if (todos.length === 0) {
    return (
      <div className="text-center py-8 text-muted-foreground">
        No todos yet. Add one above!
      </div>
    )
  }

  return (
    <div className="space-y-2">
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </div>
  )
}
```

**Yang terjadi:**

**Props `initialTodos`** → data todos yang diambil di server (`app/todos/page.tsx`) dikirim sebagai props. Ini adalah data **awal** saat halaman pertama kali di-render.

**`useState<Todo[]>(initialTodos)`** → state lokal untuk menyimpan daftar todos. Mulai dari `initialTodos`, tapi bisa berubah saat ada update dari realtime.

**`createClient()`** dari `lib/supabase/client.ts` → bikin Supabase client versi **browser**.

**Realtime subscription:**
```ts
useEffect(() => {
  const channel = supabase
    .channel('realtime todos')
    .on('postgres_changes', { event: '*', schema: 'public', table: 'todos' }, (payload) => {
      // Handle INSERT, UPDATE, DELETE
    })
    .subscribe()

  return () => { supabase.removeChannel(channel) }
}, [supabase])
```

- `useEffect` → side effect di React, jalan setelah komponen di-mount
- `supabase.channel('realtime todos')` → bikin channel WebSocket untuk dengarkan perubahan di database
- `.on('postgres_changes', ...)` → dengarkan semua event (`INSERT`, `UPDATE`, `DELETE`) di tabel `todos`
- Saat ada event → update state `todos` sesuai event type
- `.subscribe()` → mulai mendengarkan
- **Cleanup function** `() => { supabase.removeChannel(channel) }` → berhenti mendengarkan saat komponen di-unmount (mencegah memory leak)

**Hubungan dengan file lain:**
- Import `createClient` dari `lib/supabase/client.ts` → Supabase client untuk browser
- Import `<TodoItem>` dari `components/todo-item.tsx` → render setiap item todo
- Import type `Database` dari `types/database.ts` → TypeScript types dari schema Supabase
- Menerima props `initialTodos` dari `app/todos/page.tsx`

---

### 9. `components/todo-item.tsx` — Satu Item Todo

**Apa ini?** Komponen untuk menampilkan satu todo — dengan checkbox (toggle) dan tombol delete.

```tsx
'use client'

import { toggleTodo, deleteTodo } from '@/app/actions'
import { Button } from '@/components/ui/button'
import { Card, CardContent } from '@/components/ui/card'
import { Checkbox } from '@/components/ui/checkbox'
import { Trash2 } from 'lucide-react'
import type { Database } from '@/types/database'

type Todo = Database['public']['Tables']['todos']['Row']

export function TodoItem({ todo }: { todo: Todo }) {
  return (
    <Card>
      <CardContent className="flex items-center justify-between p-4">
        <div className="flex items-center gap-3">
          <Checkbox
            checked={todo.is_complete || false}
            onCheckedChange={() => toggleTodo(todo.id, todo.is_complete || false)}
          />
          <span className={todo.is_complete ? 'line-through text-muted-foreground' : ''}>
            {todo.title}
          </span>
        </div>
        <Button variant="ghost" size="icon" onClick={() => deleteTodo(todo.id)}>
          <Trash2 className="h-4 w-4" />
        </Button>
      </CardContent>
    </Card>
  )
}
```

**Yang terjadi:**

**Props `todo`** → satu object todo dengan property: `id`, `title`, `is_complete`, `created_at`, `updated_at`.

**Checkbox:**
- `checked={todo.is_complete || false}` → centang kalau `is_complete` true
- `onCheckedChange={() => toggleTodo(todo.id, todo.is_complete || false)}` → saat diklik, panggil Server Action `toggleTodo`

**Tombol Delete:**
- `onClick={() => deleteTodo(todo.id)}` → saat diklik, panggil Server Action `deleteTodo`

**Styling:**
- `line-through text-muted-foreground` → kalau todo selesai, teks dicoret dan warnanya pudar

**Hubungan dengan file lain:**
- Import `toggleTodo`, `deleteTodo` dari `app/actions.ts` → Server Actions untuk update/delete
- Import komponen UI dari `components/ui/` → Button, Card, Checkbox
- Import icon `Trash2` dari `lucide-react` → icon tempat sampah
- Import type `Database` dari `types/database.ts` → type definition untuk Todo

---

## 🔗 Ringkasan: Bagaimana Semua File Saling Terhubung

```
┌─────────────────────────────────────────────────────────────────────┐
│                           USER (Browser)                             │
│                                                                      │
│  Buka http://localhost:3000                                          │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  app/page.tsx (Server Component)                                     │
│  → redirect('/todos')                                                │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  app/todos/page.tsx (Server Component)                               │
│                                                                      │
│  1. import createClient → lib/supabase/server.ts                     │
│  2. Query: supabase.from('todos').select('*')                        │
│  3. Render UI:                                                       │
│     - <AddTodoForm /> → components/add-todo-form.tsx                 │
│     - <TodoList initialTodos={todos} /> → components/todo-list.tsx   │
└───────┬─────────────────────┬───────────────────────────────────────┘
        │                     │
        ▼                     ▼
┌───────────────────┐ ┌───────────────────────────────────────────────┐
│ add-todo-form.tsx │ │ todo-list.tsx (Client Component)              │
│ ('use client')    │ │                                                │
│                   │ │ 1. Terima initialTodos dari props              │
│ - Form submit     │ │ 2. useState untuk simpan todos                 │
│   → addTodo()     │ │ 3. useEffect → realtime subscription           │
│   (Server Action) │ │    → createClient() dari lib/supabase/client.ts│
│                   │ │ 4. Render <TodoItem> untuk setiap todo         │
│ - Import dari:    │ │                                                │
│   app/actions.ts  │ │ - Import TodoItem dari todo-item.tsx           │
└───────────────────┘ └───────────────┬───────────────────────────────┘
                                      │
                                      ▼
                          ┌───────────────────────┐
                          │ todo-item.tsx          │
                          │ ('use client')         │
                          │                        │
                          │ - Checkbox → toggleTodo│
                          │ - Button → deleteTodo  │
                          │                        │
                          │ - Import dari:         │
                          │   app/actions.ts       │
                          └───────────┬────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────┐
│  app/actions.ts (Server Actions — 'use server')                      │
│                                                                      │
│  Fungsi yang dipanggil dari client:                                  │
│  - addTodo(formData) → INSERT ke database                            │
│  - toggleTodo(id, isComplete) → UPDATE di database                   │
│  - deleteTodo(id) → DELETE dari database                             │
│                                                                      │
│  Semua fungsi:                                                       │
│  1. import createClient → lib/supabase/server.ts                     │
│  2. Query ke Supabase                                                │
│  3. revalidatePath('/todos') → refresh cache halaman                 │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  lib/supabase/                                                       │
│                                                                      │
│  server.ts → createClient() untuk Server Components & Server Actions │
│  client.ts → createClient() untuk Client Components (realtime)       │
└───────────────────────────┬─────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────────┐
│  SUPABASE (PostgreSQL Database)                                      │
│                                                                      │
│  Tabel: todos                                                        │
│  - id (UUID, primary key)                                            │
│  - title (text)                                                      │
│  - is_complete (boolean, default false)                              │
│  - created_at, updated_at (timestamp)                                │
│                                                                      │
│  RLS: DISABLED (untuk development tanpa auth)                        │
│  Realtime: ENABLED (untuk update UI tanpa refresh)                   │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🗄️ Database Schema

Tabel `todos` di Supabase:

```sql
CREATE TABLE todos (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  is_complete BOOLEAN DEFAULT false,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- RLS disabled untuk development tanpa auth
-- Realtime enabled untuk update UI secara otomatis
```

**Catatan:** Kolom `user_id` sudah dihapus dan di-set nullable karena kita tidak pakai auth.

---

## 🔑 Environment Variables

File `.env.local`:

```env
NEXT_PUBLIC_SUPABASE_URL=https://xxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

- `NEXT_PUBLIC_` prefix → variabel ini juga tersedia di browser (client-side)
- URL & API key dari Supabase dashboard → Project Settings → API

---

## 🚀 Cara Menjalankan Aplikasi

```bash
# 1. Install dependencies (jika belum)
npm install

# 2. Jalankan development server
npm run dev

# 3. Buka browser
# http://localhost:3000 → otomatis redirect ke /todos
```

---

## 🧪 Coba Sendiri

1. **Tambah todo** → ketik di input → klik "Add Todo" → muncul di list
2. **Toggle todo** → klik checkbox → teks dicoret (selesai)
3. **Delete todo** → klik icon tempat sampah → todo hilang
4. **Buka 2 tab browser** → tambah todo di satu tab → lihat muncul di tab lain (realtime!)

---

## 📚 Glosarium

| Istilah | Penjelasan |
|---|---|
| **Server Component** | Komponen yang jalan di server. Bisa query database langsung. Tidak bisa pakai React hooks. |
| **Client Component** | Komponen yang jalan di browser. Bisa pakai `useState`, `useEffect`, dll. Harus ada `'use client'`. |
| **Server Action** | Fungsi yang jalan di server. Dipanggil dari client via form action atau function call. Ditandai `'use server'`. |
| **App Router** | Sistem routing Next.js berdasarkan struktur folder di `app/`. |
| **Revalidation** | Proses refresh cache di Next.js. `revalidatePath()` bilang Next.js untuk ambil data terbaru. |
| **Realtime** | Fitur Supabase yang mengirim event via WebSocket saat ada perubahan di database. |
| **RLS (Row Level Security)** | Fitur keamanan Supabase yang membatasi akses data per user. Kita disable untuk development tanpa auth. |
| **Shadcn UI** | Koleksi komponen UI yang bisa di-copy ke proyek. Bukan library yang di-install. |

---

## 🎯 Kesimpulan

Aplikasi ini memiliki arsitektur sebagai berikut:

1. **Frontend (Client Components)** → `components/*.tsx` → UI interaktif dengan React hooks
2. **Backend (Server Components + Server Actions)** → `app/*.tsx`, `app/actions.ts` → query database & logic
3. **Database** → Supabase (PostgreSQL) → menyimpan data todos
4. **Realtime** → Supabase WebSocket → update UI otomatis saat ada perubahan

**Komunikasi antar layer:**
- Client → Server: lewat **Server Actions** (`addTodo`, `toggleTodo`, `deleteTodo`)
- Server → Database: lewat **Supabase client** (`supabase.from('todos')...`)
- Database → Client: lewat **Realtime subscription** (WebSocket)
- Server → Client: lewat **props** (data awal) dan **revalidation** (refresh cache)

Selamat belajar! 🚀

Tambahan :
Mari kita bedah secara mendalam apa yang sebenarnya terjadi di balik layar Next.js.
## 1. SSR (Server-Side Rendering) di Next.js
Bayangkan kamu memesan makanan di restoran dan pelayan membawakan piring yang sudah berisi nasi dan lauk matang. Kamu tinggal makan.

* Apa yang dikirim dari Server ke Client?
1. HTML Matang: Dokumen HTML lengkap yang sudah berisi data (misalnya nama user dari database). Jika kamu "View Page Source", kamu bisa baca semua tulisannya.
   2. CSS: File gaya agar tampilan langsung rapi.
   3. JSON Data: Data yang dipakai server untuk merender HTML tadi (untuk sinkronisasi).
   4. Hydration Script: Bundle JavaScript kecil untuk membuat halaman jadi interaktif (misal: fungsi klik tombol).
* Apa yang diolah Server?
* Membaca Cookies (untuk cek login).
   * Mengambil data dari Database (menggunakan fetch atau ORM).
   * Menyusun komponen React menjadi teks HTML.
* Apa yang diolah Client (Browser)?
* Parsing HTML: Langsung menampilkan teks dan gambar (sangat cepat).
   * Hydration: Menghubungkan JavaScript ke HTML yang sudah ada supaya tombol-tombol bisa diklik.

------------------------------
## 2. CSR (Client-Side Rendering) di Next.js
(Biasanya menggunakan direktori "use client")
Ini ibarat kamu memesan makanan, tapi dikirim bahan mentahnya saja. Kamu harus masak sendiri di meja makan.

* Apa yang dikirim dari Server ke Client?
1. HTML Kosong/Shell: Cuma kerangka dasar (biasanya cuma ada tag <body> kosong).
   2. Bundle JavaScript Raksasa: Isinya seluruh logika React, library, dan instruksi bagaimana cara membuat tampilan.
* Apa yang diolah Server?
* Hampir tidak ada. Server cuma jadi "gudang file" yang mengirimkan file .js.
* Apa yang diolah Client (Browser)?
* Download & Execute JS: Browser harus download file JS yang besar, lalu menjalankannya.
   * Data Fetching: Baru setelah JS jalan, browser sadar dia butuh data, lalu melakukan request tambahan ke API (pake token dari LocalStorage atau Cookie).
   * Rendering: Browser membangun elemen HTML satu per satu di memori laptop/HP kamu.

------------------------------
## 3. Apakah CSR di Framework Lain Sama?
Pada dasarnya SAMA. Cara kerja CSR di Next.js (pada komponen "use client") identik dengan cara kerja React murni (Vite/CRA), Vue, atau Angular.
Perbedaannya adalah:

* React Biasa/Vite: Secara default HANYA bisa CSR. Semuanya diolah di browser.
* Next.js: Dia adalah framework Hybrid. Secara default dia akan mencoba SSR dulu (Server Components). Kamu baru pakai CSR kalau butuh interaksi browser (seperti useState, useEffect, atau akses sensor kamera).

## Ringkasan Bundle yang Dikirim:

| Item | SSR (Next.js) | CSR (Framework Lain / use client) |
|---|---|---|
| HTML | Isi konten lengkap (Seo Friendly) | Tag kosong (Tidak terbaca Google Bot) |
| JS Size | Lebih kecil (Hanya untuk interaksi) | Sangat besar (Berisi seluruh logika web) |
| Data | Sudah tertempel di HTML | Harus diambil lagi via API (Loading) |
| Akses Simpanan | Bisa baca Cookie di server | Harus baca LocalStorage di browser |

Kesimpulan: Next.js menang karena dia mengirimkan "hasil jadi" (SSR), sedangkan CSR di framework lain mengirimkan "instruksi kerja" yang bikin browser user harus kerja lebih keras.
Mau saya jelaskan lebih detail tentang kenapa Next.js membagi JS menjadi potongan kecil (Code Splitting) agar bundle-nya tidak berat?


