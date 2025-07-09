# Rest Express - AI-Powered E-commerce Store Builder

## Overview

This is a full-stack e-commerce platform that enables users to create and manage online stores with AI-powered assistance. The application uses a modern tech stack with React/Vite frontend, Express.js backend, and PostgreSQL database. The system features AI-driven product generation, store customization, and comprehensive e-commerce functionality.

## System Architecture

### Frontend Architecture
- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite for fast development and optimized builds
- **Routing**: Wouter for lightweight client-side routing
- **State Management**: TanStack Query (React Query) for server state management
- **Styling**: Tailwind CSS with shadcn/ui component library
- **UI Components**: Radix UI primitives with custom styling
- **Form Handling**: React Hook Form with Zod validation
- **Internationalization**: RTL support with Arabic fonts (Noto Sans Arabic)

### Backend Architecture
- **Framework**: Express.js with TypeScript
- **Database ORM**: Drizzle ORM for type-safe database operations
- **Database**: PostgreSQL (configured for Neon serverless)
- **API Design**: RESTful API endpoints
- **Session Management**: Express sessions with PostgreSQL storage

### Data Storage Solutions
- **Primary Database**: PostgreSQL with Drizzle ORM
- **Schema Management**: Drizzle Kit for migrations
- **Fallback Storage**: In-memory storage implementation for development
- **Session Storage**: PostgreSQL-backed session store

## Key Components

### Database Schema
- **Stores Table**: Core store information including name, category, settings, and custom domains
- **Products Table**: Product catalog with AI generation flags and inventory tracking
- **Orders Table**: Order management with customer details and item arrays
- **JSON Fields**: Flexible settings storage for store customization and order items

### AI Integration
- **OpenAI Integration**: GPT-4o model for content generation
- **Product Description Generation**: AI-powered product content creation
- **Pricing Suggestions**: Intelligent pricing recommendations
- **Image Generation**: AI-generated product images
- **Store Optimization**: AI-enhanced store descriptions

### Frontend Pages
- **Home Page**: Landing page with feature showcase and templates
- **Store Creation Wizard**: Multi-step store setup process
- **Dashboard**: Store management interface with analytics
- **Store View**: Customer-facing storefront
- **Product Management**: CRUD operations for products

### UI/UX Design
- **Design System**: shadcn/ui with custom SooqBot branding
- **Responsive Design**: Mobile-first approach
- **Arabic Support**: RTL layout with proper Arabic typography
- **Accessibility**: ARIA-compliant components
- **Dark Mode**: CSS variable-based theming support

## Data Flow

1. **Store Creation**: Users go through a wizard to create stores with AI assistance
2. **Product Management**: Store owners can manually add products or use AI generation
3. **AI Processing**: OpenAI API generates product descriptions, images, and pricing
4. **Database Operations**: All data persisted through Drizzle ORM
5. **Customer Experience**: Public store views with shopping cart functionality
6. **Order Processing**: Complete order management system

## External Dependencies

### Core Framework Dependencies
- React ecosystem (React, React DOM, React Hook Form)
- Vite build toolchain with TypeScript support
- Express.js server framework

### Database & ORM
- Drizzle ORM with PostgreSQL dialect
- Neon Database serverless PostgreSQL
- Connection pooling and session management

### AI Services
- OpenAI API for content generation
- Image generation capabilities
- Natural language processing

### UI Libraries
- Radix UI component primitives
- Tailwind CSS for styling
- Lucide React for icons
- Custom fonts (Noto Sans Arabic, Inter)

### Development Tools
- TypeScript for type safety
- ESBuild for server bundling
- PostCSS for CSS processing
- Replit-specific development plugins

## Deployment Strategy

### Build Process
- **Frontend**: Vite builds optimized static assets to `dist/public`
- **Backend**: ESBuild bundles server code to `dist/index.js`
- **Database**: Drizzle migrations applied via `db:push` command

### Environment Configuration
- **Development**: Hot reloading with Vite dev server
- **Production**: Express serves static files and API endpoints
- **Database**: Environment-based DATABASE_URL configuration

### Hosting Considerations
- Single server deployment (Express serves both API and static files)
- PostgreSQL database connection via connection string
- Environment variables for API keys and database credentials

## Changelog

```
Changelog:
- June 27, 2025. Initial setup
- June 29, 2025. Added PostgreSQL database with full DatabaseStorage implementation
- June 29, 2025. Created comprehensive mobile conversion guide for React Native
- June 29, 2025. Added production deployment guide for multiple platforms
- June 29, 2025. Documented complete component architecture and data flow
- July 01, 2025. Added bilingual support (Arabic/English) and BHD currency
- July 01, 2025. Updated all pricing displays to use Bahraini Dinars (BHD)
- July 01, 2025. Fixed routing system and made all application links functional
- July 01, 2025. Created mobile-optimized PWA with Arabic RTL support
```

## User Preferences

```
Preferred communication style: Simple, everyday language.
Currency: BHD (Bahraini Dinars)
Languages: Arabic (primary) and English support
Mobile compatibility: Essential for PWA usage
```