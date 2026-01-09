# Frontend Technology - Next.js (React)

Complete guide to Next.js frontend implementation for DAPK system.

---

## Next.js Overview

**Official Site**: https://nextjs.org
**GitHub**: https://github.com/vercel/next.js (115,000+ stars)
**License**: MIT

Next.js is a React framework for building full-stack web applications with server-side rendering, static site generation, and excellent developer experience.

---

## Why Next.js for DAPK?

### SEO Optimization
- **Server-Side Rendering (SSR)**: Pages rendered on server
- **Static Site Generation (SSG)**: Pre-render at build time
- Critical for corporate website to rank in search engines

### Performance
- **Automatic Code Splitting**: Load only what's needed
- **Image Optimization**: Next/Image component
- **Fast Refresh**: See changes instantly
- **Incremental Static Regeneration**: Update static pages without rebuilding

### Developer Experience
- **File-based Routing**: No router configuration
- **TypeScript Support**: Built-in
- **API Routes**: Optional backend endpoints
- **Hot Module Replacement**: Instant updates

---

## Project Structure

```
frontend/
├── public/                        # Static files
│   ├── images/
│   ├── favicon.ico
│   └── robots.txt
│
├── src/
│   ├── app/                       # App Router (Next.js 14)
│   │   ├── layout.tsx             # Root layout
│   │   ├── page.tsx               # Homepage
│   │   ├── loading.tsx            # Loading UI
│   │   ├── error.tsx              # Error UI
│   │   │
│   │   ├── (auth)/                # Auth group
│   │   │   ├── login/
│   │   │   │   └── page.tsx       # Login page
│   │   │   └── register/
│   │   │       └── page.tsx       # Register page
│   │   │
│   │   ├── dashboard/             # Dashboard (authenticated)
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx
│   │   │   ├── shipments/
│   │   │   │   ├── page.tsx       # List shipments
│   │   │   │   ├── create/
│   │   │   │   │   └── page.tsx   # Create shipment
│   │   │   │   └── [id]/
│   │   │   │       └── page.tsx   # Shipment details
│   │   │   ├── manifests/
│   │   │   ├── runsheets/
│   │   │   ├── cash-register/
│   │   │   └── reports/
│   │   │
│   │   ├── tracking/              # Public tracking
│   │   │   └── [awb]/
│   │   │       └── page.tsx
│   │   │
│   │   ├── about/
│   │   ├── services/
│   │   └── contact/
│   │
│   ├── components/                # Reusable components
│   │   ├── ui/                    # Base UI components
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Table.tsx
│   │   │   └── Card.tsx
│   │   ├── layout/
│   │   │   ├── Header.tsx
│   │   │   ├── Sidebar.tsx
│   │   │   ├── Footer.tsx
│   │   │   └── Navigation.tsx
│   │   ├── forms/
│   │   │   ├── ShipmentForm.tsx
│   │   │   ├── LoginForm.tsx
│   │   │   └── ManifestForm.tsx
│   │   └── features/
│   │       ├── ShipmentList.tsx
│   │       ├── TrackingWidget.tsx
│   │       └── StatusBadge.tsx
│   │
│   ├── lib/                       # Utilities & helpers
│   │   ├── api/                   # API client
│   │   │   ├── client.ts          # Axios instance
│   │   │   ├── auth.ts            # Auth API calls
│   │   │   ├── shipments.ts       # Shipment API calls
│   │   │   └── ...
│   │   ├── hooks/                 # Custom React hooks
│   │   │   ├── useAuth.ts
│   │   │   ├── useShipments.ts
│   │   │   └── useDebounce.ts
│   │   ├── utils/
│   │   │   ├── formatters.ts      # Date, currency formatters
│   │   │   ├── validators.ts      # Form validators
│   │   │   └── constants.ts
│   │   └── types/
│   │       ├── api.ts             # API response types
│   │       ├── models.ts          # Data models
│   │       └── forms.ts           # Form types
│   │
│   ├── store/                     # State management
│   │   ├── authStore.ts           # Auth state (Zustand)
│   │   ├── shipmentStore.ts       # Shipment state
│   │   └── uiStore.ts             # UI state
│   │
│   └── styles/
│       └── globals.css            # Global styles
│
├── .env.local                     # Environment variables
├── .env.example
├── next.config.js                 # Next.js configuration
├── tailwind.config.js             # Tailwind CSS config
├── tsconfig.json                  # TypeScript config
├── package.json
└── Dockerfile
```

---

## Key Dependencies

```json
{
  "dependencies": {
    // Framework
    "next": "14.0.4",
    "react": "18.2.0",
    "react-dom": "18.2.0",
    
    // Routing & Navigation
    "next/navigation": "Built-in",
    
    // HTTP Client
    "axios": "1.6.2",
    
    // State Management
    "zustand": "4.4.7",
    
    // Form Handling
    "react-hook-form": "7.49.2",
    "zod": "3.22.4",
    "@hookform/resolvers": "3.3.3",
    
    // UI Components
    "tailwindcss": "3.3.6",
    "@headlessui/react": "1.7.17",
    "@heroicons/react": "2.1.1",
    
    // Date & Time
    "date-fns": "2.30.0",
    
    // Utilities
    "clsx": "2.0.0",
    "lodash": "4.17.21"
  },
  "devDependencies": {
    "typescript": "5.3.3",
    "@types/node": "20.10.6",
    "@types/react": "18.2.46",
    "eslint": "8.55.0",
    "eslint-config-next": "14.0.4",
    "prettier": "3.1.1",
    "autoprefixer": "10.4.16",
    "postcss": "8.4.32"
  }
}
```

---

## Configuration

### next.config.js

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Output standalone for Docker
  output: 'standalone',
  
  // API base URL from environment
  env: {
    NEXT_PUBLIC_API_URL: process.env.NEXT_PUBLIC_API_URL,
  },
  
  // Image optimization
  images: {
    domains: ['api.dapk.com'],
    formats: ['image/webp', 'image/avif'],
  },
  
  // Disable X-Powered-By header
  poweredByHeader: false,
  
  // Strict mode
  reactStrictMode: true,
  
  // Experimental features
  experimental: {
    typedRoutes: true,
  },
}

module.exports = nextConfig
```

### tailwind.config.js

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
        },
      },
    },
  },
  plugins: [
    require('@tailwindcss/forms'),
    require('@tailwindcss/typography'),
  ],
}
```

---

## API Client Setup

### src/lib/api/client.ts

```typescript
import axios, { AxiosInstance, AxiosError } from 'axios';

const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8000/v1';

// Create axios instance
const apiClient: AxiosInstance = axios.create({
  baseURL: API_BASE_URL,
  headers: {
    'Content-Type': 'application/json',
  },
  timeout: 30000, // 30 seconds
});

// Request interceptor (add auth token)
apiClient.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('access_token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// Response interceptor (handle errors)
apiClient.interceptors.response.use(
  (response) => response,
  async (error: AxiosError) => {
    if (error.response?.status === 401) {
      // Token expired, try refresh
      const refreshToken = localStorage.getItem('refresh_token');
      if (refreshToken) {
        try {
          const response = await axios.post(`${API_BASE_URL}/auth/refresh`, {
            refresh_token: refreshToken,
          });
          
          const newToken = response.data.access_token;
          localStorage.setItem('access_token', newToken);
          
          // Retry original request
          if (error.config) {
            error.config.headers.Authorization = `Bearer ${newToken}`;
            return axios(error.config);
          }
        } catch (refreshError) {
          // Refresh failed, redirect to login
          localStorage.removeItem('access_token');
          localStorage.removeItem('refresh_token');
          window.location.href = '/login';
        }
      }
    }
    
    return Promise.reject(error);
  }
);

export default apiClient;
```

---

## State Management (Zustand)

### src/store/authStore.ts

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface User {
  id: string;
  email: string;
  full_name: string;
  role: string;
}

interface AuthState {
  user: User | null;
  accessToken: string | null;
  isAuthenticated: boolean;
  
  login: (user: User, accessToken: string) => void;
  logout: () => void;
  updateUser: (user: User) => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      accessToken: null,
      isAuthenticated: false,
      
      login: (user, accessToken) => {
        localStorage.setItem('access_token', accessToken);
        set({ user, accessToken, isAuthenticated: true });
      },
      
      logout: () => {
        localStorage.removeItem('access_token');
        localStorage.removeItem('refresh_token');
        set({ user: null, accessToken: null, isAuthenticated: false });
      },
      
      updateUser: (user) => set({ user }),
    }),
    {
      name: 'auth-storage',
    }
  )
);
```

---

## Custom Hooks

### src/lib/hooks/useAuth.ts

```typescript
'use client';

import { useEffect } from 'react';
import { useRouter } from 'next/navigation';
import { useAuthStore } from '@/store/authStore';

export function useAuth(requireAuth: boolean = true) {
  const router = useRouter();
  const { user, isAuthenticated, logout } = useAuthStore();
  
  useEffect(() => {
    if (requireAuth && !isAuthenticated) {
      router.push('/login');
    }
  }, [isAuthenticated, requireAuth, router]);
  
  return {
    user,
    isAuthenticated,
    logout,
  };
}
```

### src/lib/hooks/useShipments.ts

```typescript
'use client';

import { useState, useEffect } from 'react';
import { getShipments } from '@/lib/api/shipments';
import type { Shipment } from '@/lib/types/models';

export function useShipments(filters?: any) {
  const [shipments, setShipments] = useState<Shipment[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    async function fetchShipments() {
      try {
        setLoading(true);
        const data = await getShipments(filters);
        setShipments(data.data);
      } catch (err: any) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    
    fetchShipments();
  }, [filters]);
  
  return { shipments, loading, error };
}
```

---

## Form Handling (React Hook Form + Zod)

### src/components/forms/ShipmentForm.tsx

```typescript
'use client';

import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { createShipment } from '@/lib/api/shipments';

// Validation schema
const shipmentSchema = z.object({
  sender_name: z.string().min(1, 'Required'),
  sender_phone: z.string().regex(/^\+?[0-9]{10,15}$/, 'Invalid phone'),
  sender_address: z.string().min(1, 'Required'),
  receiver_name: z.string().min(1, 'Required'),
  receiver_phone: z.string().regex(/^\+?[0-9]{10,15}$/, 'Invalid phone'),
  receiver_address: z.string().min(1, 'Required'),
  weight_kg: z.number().positive('Must be positive'),
  commodity: z.string().min(1, 'Required'),
  service_type: z.enum(['regular', 'express']),
});

type ShipmentFormData = z.infer<typeof shipmentSchema>;

export function ShipmentForm() {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<ShipmentFormData>({
    resolver: zodResolver(shipmentSchema),
  });
  
  const onSubmit = async (data: ShipmentFormData) => {
    try {
      const result = await createShipment(data);
      alert(`Shipment created! AWB: ${result.awb_number}`);
    } catch (error: any) {
      alert(`Error: ${error.message}`);
    }
  };
  
  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-6">
      <div>
        <label className="block text-sm font-medium">Sender Name</label>
        <input
          {...register('sender_name')}
          className="mt-1 block w-full rounded-md border-gray-300"
        />
        {errors.sender_name && (
          <p className="mt-1 text-sm text-red-600">{errors.sender_name.message}</p>
        )}
      </div>
      
      {/* More fields... */}
      
      <button
        type="submit"
        disabled={isSubmitting}
        className="w-full bg-blue-600 text-white py-2 px-4 rounded-md hover:bg-blue-700"
      >
        {isSubmitting ? 'Creating...' : 'Create Shipment'}
      </button>
    </form>
  );
}
```

---

## Page Examples

### src/app/dashboard/shipments/page.tsx

```typescript
'use client';

import { useShipments } from '@/lib/hooks/useShipments';
import { useAuth } from '@/lib/hooks/useAuth';
import { ShipmentList } from '@/components/features/ShipmentList';

export default function ShipmentsPage() {
  useAuth(); // Require authentication
  
  const { shipments, loading, error } = useShipments();
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-6">Shipments</h1>
      <ShipmentList shipments={shipments} />
    </div>
  );
}
```

### src/app/tracking/[awb]/page.tsx

```typescript
import { getPublicTracking } from '@/lib/api/tracking';
import { TrackingWidget } from '@/components/features/TrackingWidget';

// This is a Server Component (no 'use client')
export default async function TrackingPage({ 
  params 
}: { 
  params: { awb: string } 
}) {
  const tracking = await getPublicTracking(params.awb);
  
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-3xl font-bold mb-6">Track Shipment</h1>
      <TrackingWidget data={tracking} />
    </div>
  );
}
```

---

## Styling with Tailwind CSS

### Component Example

```typescript
export function Button({ 
  children, 
  variant = 'primary', 
  ...props 
}: ButtonProps) {
  const baseClasses = 'px-4 py-2 rounded-md font-medium transition-colors';
  
  const variants = {
    primary: 'bg-blue-600 text-white hover:bg-blue-700',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300',
    danger: 'bg-red-600 text-white hover:bg-red-700',
  };
  
  return (
    <button
      className={`${baseClasses} ${variants[variant]}`}
      {...props}
    >
      {children}
    </button>
  );
}
```

---

## Running the Application

### Development

```bash
# Install dependencies
npm install

# Run dev server
npm run dev

# Open http://localhost:3000
```

### Production Build

```bash
# Build for production
npm run build

# Start production server
npm start
```

### Docker

```dockerfile
# Dockerfile
FROM node:20-alpine AS base

# Dependencies
FROM base AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Builder
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Runner
FROM base AS runner
WORKDIR /app
ENV NODE_ENV=production

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
CMD ["node", "server.js"]
```

---

## Performance Optimization

### Image Optimization

```typescript
import Image from 'next/image';

<Image
  src="/images/logo.png"
  alt="DAPK Logo"
  width={200}
  height={50}
  priority // Load immediately (above fold)
/>
```

### Code Splitting

```typescript
// Dynamic import for heavy components
import dynamic from 'next/dynamic';

const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  loading: () => <p>Loading chart...</p>,
  ssr: false, // Disable server-side rendering
});
```

### Data Fetching

```typescript
// Server Component (SSR)
async function getData() {
  const res = await fetch('https://api.dapk.com/v1/shipments', {
    next: { revalidate: 60 }, // Revalidate every 60 seconds
  });
  return res.json();
}

export default async function Page() {
  const data = await getData();
  return <div>{/* Render data */}</div>;
}
```

---

## Testing

### Unit Tests (Jest + React Testing Library)

```typescript
import { render, screen } from '@testing-library/react';
import { Button } from '@/components/ui/Button';

describe('Button', () => {
  it('renders button with text', () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText('Click me')).toBeInTheDocument();
  });
  
  it('handles click events', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click</Button>);
    screen.getByText('Click').click();
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});
```

### E2E Tests (Cypress)

```typescript
describe('Shipment Creation', () => {
  it('creates a new shipment', () => {
    cy.visit('/dashboard/shipments/create');
    
    cy.get('input[name="sender_name"]').type('John Doe');
    cy.get('input[name="receiver_name"]').type('Jane Smith');
    // Fill other fields...
    
    cy.get('button[type="submit"]').click();
    
    cy.contains('Shipment created').should('be.visible');
  });
});
```

---

## Related Documentation

- [Tech Stack Overview](overview.md)
- [Backend FastAPI](backend-fastapi.md)
- [System Architecture](../01-architecture/system-architecture.md)
- [API Specification](../04-api/openapi-spec.yaml)

