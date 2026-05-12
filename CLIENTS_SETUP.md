Client index page (Option A - per-page React entrypoint)

Copy and paste each section into the indicated file in your Rails project.

1) Add route (edit `config/routes.rb` inside the `Rails.application.routes.draw do` block)

```ruby
# add this near other top-level routes
get 'clients', to: 'clients#index'
```

2) Create controller — file: `app/controllers/clients_controller.rb`

```ruby
class ClientsController < ApplicationController
  def index
    # Optional: load server data if you want to embed it into the view:
    # @clients = Client.all
  end
end
```

3) Create Rails view — file: `app/views/clients/index.html.erb`

```erb
<div id="clients-root"></div>

<%= vite_javascript_tag 'clients_index.tsx' %>
```

4) Create React entrypoint — file: `app/frontend/entrypoints/clients_index.tsx`

```tsx
import React from 'react'
import { createRoot } from 'react-dom/client'
import ClientsIndex from '../components/ClientsIndex'

const el = document.getElementById('clients-root')
if (el) {
  createRoot(el).render(<ClientsIndex />)
}
```

5) Create React component — file: `app/frontend/components/ClientsIndex.tsx`

```tsx
import React, { useEffect, useState } from 'react'

type Client = { id: number; name: string }

export default function ClientsIndex() {
  const [clients, setClients] = useState<Client[]>([])

  useEffect(() => {
    fetch('/api/v1/clients')
      .then(res => res.json())
      .then(setClients)
      .catch(err => {
        console.error('Failed to load clients', err)
      })
  }, [])

  return (
    <div style={{ padding: 20 }}>
      <h1>Clients</h1>
      <ul>
        {clients.map(c => <li key={c.id}>{c.name}</li>)}
      </ul>
    </div>
  )
}
```

6) Install TypeScript & React types (if not already)

```bash
# from project root (oi_backend)
npm install --save-dev typescript @types/react @types/react-dom
```

7) Ensure `tsconfig.json` contains at least:

```jsonc
{
  "compilerOptions": {
    "module": "nodenext",
    "target": "esnext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,
    "sourceMap": true,
    "isolatedModules": true,
    "types": ["vite/client"]
  }
}
```

8) Start dev servers

```bash
# recommended (uses Procfile.dev)
bin/dev

# or run separately
bin/vite dev
bin/rails s
```

9) Verify

- Open: http://localhost:3000/clients
- API test:

```bash
curl http://localhost:3000/api/v1/clients
```

Notes
- If `Client` model or `/api/v1/clients` API doesn't exist, generate model and implement the namespaced API controller first (prefer `rails g model` and a manual `Api::V1::ClientsController`).
- To render initial server data without an extra fetch, set `@clients` in controller and embed JSON in the view:

```erb
<script id="initial-clients" type="application/json"><%= raw @clients.to_json %></script>
```

Then read it from React:

```ts
const initial = JSON.parse(document.getElementById('initial-clients')!.textContent || '[]')
```

If you want, I can create these files for you — tell me to proceed and I will add them to the repo.
