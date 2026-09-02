# CloudRunner Frontend (SvelteKit + Deno)

A modern frontend application built with SvelteKit and Deno for managing NixOS configurations.

## 🚀 Features

- **Authentication**
  - Login/Register
  - Password reset
  - JWT token management
  - Auto-redirect on authentication state

- **NixOS Configuration Management**
  - Upload new configurations
  - View all configurations
  - Edit existing configurations
  - Delete configurations
  - Real-time file size display

- **Modern UI**
  - Responsive design with Tailwind CSS
  - Modal dialogs for upload/edit
  - Loading states
  - Error handling

## 📋 Prerequisites

- Deno 1.40+
- Backend server running (Axum backend)

## 🎯 Setup Instructions

### 1. Install Dependencies

```bash
# Install npm packages via Deno
deno cache --reload npm:vite npm:svelte npm:@sveltejs/kit
```

### 2. Configure Environment

```bash
# Copy environment template
cp .env.example .env

# Edit .env if your backend is on a different URL
# Default: http://localhost:8080
```

### 3. Install Tailwind CSS

```bash
# Install Tailwind and dependencies
deno run -A npm:tailwindcss init -p
```

### 4. Run Development Server

```bash
deno task dev
```

The app will be available at `http://localhost:5173`

## 📁 Project Structure

```
cloud-frontend/
├── src/
│   ├── lib/
│   │   ├── api/
│   │   │   └── client.ts       # API client with typed methods
│   │   └── stores/
│   │       └── auth.ts         # Authentication store
│   ├── routes/
│   │   ├── +layout.svelte      # Root layout (imports CSS)
│   │   ├── +page.svelte        # Home (redirects)
│   │   ├── login/
│   │   │   └── +page.svelte    # Login page
│   │   ├── register/
│   │   │   └── +page.svelte    # Register page
│   │   └── dashboard/
│   │       ├── +layout.svelte  # Dashboard layout with nav
│   │       └── +page.svelte    # Main dashboard
│   ├── app.css                 # Tailwind imports
│   └── app.html                # HTML template
├── .env                        # Environment variables
├── deno.json                   # Deno configuration
├── tailwind.config.js          # Tailwind configuration
└── README.md
```

## 🔌 API Integration

The frontend connects to the Axum backend via the API client (`src/lib/api/client.ts`).

### API Configuration

Edit `.env` to change the backend URL:

```env
PUBLIC_API_URL=http://localhost:8080
VITE_API_URL=http://localhost:8080
```

### Authentication Flow

1. User logs in → Token stored in localStorage
2. Token automatically included in all API requests
3. On 401 error → Clear token and redirect to login
4. Logout → Clear token from localStorage

## 🎨 Pages

### Login (`/login`)
- Email and password authentication
- Error handling
- Links to register and password reset

### Register (`/register`)
- Full name, email, department, password
- Password confirmation
- Form validation
- Auto-redirect after successful registration

### Dashboard (`/dashboard`)
- Protected route (requires authentication)
- List all NixOS configurations
- Upload new configuration (modal)
- Edit configuration (modal)
- Delete configuration (with confirmation)
- File size and date display

## 🛠️ Development

### Available Commands

```bash
# Development server
deno task dev

# Build for production
deno task build

# Preview production build
deno task preview

# Type checking
deno task check

# Format code
deno task format
```

### Adding New Features

1. **Add API endpoint** in `src/lib/api/client.ts`
2. **Create page** in `src/routes/`
3. **Update navigation** in `src/routes/dashboard/+layout.svelte`

## 🎯 Using with Deno

This project uses Deno's npm compatibility to run SvelteKit:

```bash
# Run any npm package
deno run -A npm:<package-name>

# Example: Run vite directly
deno run -A npm:vite dev
```

## 🔐 Security

- JWT tokens stored in localStorage
- Automatic token expiration handling
- Protected routes with auth checks
- CORS handled by backend

## 🚀 Production Build

```bash
# Build the application
deno task build

# Preview the build
deno task preview

# Or use a static file server
deno run --allow-net --allow-read npm:serve build
```

## 🐛 Troubleshooting

### Backend Connection Issues

1. Make sure backend is running on `http://localhost:8080`
2. Check CORS settings in backend
3. Verify `.env` has correct API URL

### Authentication Issues

1. Clear localStorage: `localStorage.clear()`
2. Check JWT token expiration in backend
3. Verify token is being sent in headers

### Styling Issues

1. Make sure Tailwind is properly configured
2. Check that `app.css` is imported in root layout
3. Rebuild Tailwind: `deno run -A npm:tailwindcss -i ./src/app.css -o ./src/output.css`

## 📚 Tech Stack

- **Framework**: SvelteKit
- **Runtime**: Deno
- **Styling**: Tailwind CSS
- **Type Safety**: TypeScript
- **State Management**: Svelte stores
- **HTTP Client**: Fetch API

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📝 License

MIT

---

Built with ❤️ using Deno, SvelteKit, and Tailwind CSS
