# Component Architecture Guide - SooqBot

## Overview
This guide explains how each component works in your AI-powered Arabic e-commerce platform, making it easy to understand, modify, and extend the codebase.

## Project Structure Analysis

```
SooqBot/
├── client/src/                 # Frontend React Application
│   ├── components/            # Reusable UI Components
│   ├── pages/                # Main Application Pages
│   ├── lib/                  # Utilities and API Client
│   └── hooks/                # Custom React Hooks
├── server/                    # Backend Express Server
│   ├── services/             # External Service Integrations
│   ├── routes.ts             # API Route Definitions
│   ├── storage.ts            # Database Interface
│   └── db.ts                 # Database Connection
├── shared/                    # Type Definitions
│   └── schema.ts             # Database Schema & Validation
└── Configuration Files
```

## Core Components Breakdown

### 1. Database Schema (`shared/schema.ts`)

**Purpose**: Defines all data structures and validation rules

```typescript
// Three main tables with relationships:

// Stores Table - Main store information
export const stores = pgTable("stores", {
  id: serial("id").primaryKey(),
  name: text("name").notNull(),
  category: text("category").notNull(),
  description: text("description"),
  template: text("template"),
  customDomain: text("custom_domain"),
  isActive: boolean("is_active").default(true),
  createdAt: timestamp("created_at").defaultNow(),
  settings: json("settings").$type<StoreSettings>()
});

// Products Table - Product catalog
export const products = pgTable("products", {
  id: serial("id").primaryKey(),
  storeId: integer("store_id").references(() => stores.id),
  name: text("name").notNull(),
  description: text("description").notNull(),
  price: numeric("price", { precision: 10, scale: 2 }).notNull(),
  category: text("category").notNull(),
  imageUrl: text("image_url"),
  stock: integer("stock").default(0),
  isActive: boolean("is_active").default(true),
  aiGenerated: boolean("ai_generated").default(false),
  createdAt: timestamp("created_at").defaultNow()
});

// Orders Table - Customer orders
export const orders = pgTable("orders", {
  id: serial("id").primaryKey(),
  storeId: integer("store_id").references(() => stores.id),
  customerName: text("customer_name").notNull(),
  customerEmail: text("customer_email").notNull(),
  customerPhone: text("customer_phone"),
  totalAmount: numeric("total_amount", { precision: 10, scale: 2 }).notNull(),
  status: text("status").default("pending"),
  items: json("items").$type<OrderItem[]>().notNull(),
  createdAt: timestamp("created_at").defaultNow()
});
```

**Key Features**:
- **Type Safety**: Zod schemas ensure data validation
- **Relationships**: Foreign keys link stores, products, and orders
- **JSON Fields**: Flexible settings and order items storage
- **Arabic Support**: Text fields support UTF-8 Arabic characters

### 2. Storage Interface (`server/storage.ts`)

**Purpose**: Abstracts database operations with clean interface

```typescript
export interface IStorage {
  // Store operations
  createStore(store: InsertStore): Promise<Store>;
  getStore(id: number): Promise<Store | undefined>;
  listStores(): Promise<Store[]>;
  
  // Product operations
  createProduct(product: InsertProduct): Promise<Product>;
  getProductsByStore(storeId: number): Promise<Product[]>;
  
  // Order operations
  createOrder(order: InsertOrder): Promise<Order>;
  getOrdersByStore(storeId: number): Promise<Order[]>;
  
  // Analytics
  getStoreStats(storeId: number): Promise<StoreStats>;
}
```

**Implementation Pattern**:
- **Interface-based**: Easy to swap between memory and database storage
- **Async/Await**: Modern promise-based operations
- **Type-safe**: Uses TypeScript interfaces for all operations
- **Error Handling**: Graceful handling of database errors

### 3. API Routes (`server/routes.ts`)

**Purpose**: Defines REST API endpoints for frontend communication

```typescript
// Store Management
app.post('/api/stores', async (req, res) => {
  // Validates data with Zod schema
  const validatedData = storeWizardSchema.parse(req.body);
  
  // Uses storage interface
  const store = await storage.createStore(validatedData);
  
  res.json(store);
});

// AI Product Generation
app.post('/api/stores/:id/generate-products', async (req, res) => {
  const { category, count, priceRange } = req.body;
  
  // Integrates with OpenAI service
  const products = await generateProductDescriptions(category, count, priceRange);
  
  // Saves to database
  const savedProducts = await Promise.all(
    products.map(product => storage.createProduct({
      storeId: parseInt(req.params.id),
      ...product,
      aiGenerated: true
    }))
  );
  
  res.json({ products: savedProducts });
});
```

**Key Patterns**:
- **Validation First**: All input validated with Zod schemas
- **Error Handling**: Comprehensive error responses
- **Async Operations**: Non-blocking database and AI operations
- **RESTful Design**: Standard HTTP methods and status codes

### 4. AI Services (`server/services/openai.ts`)

**Purpose**: Integrates OpenAI API for content generation

```typescript
export async function generateProductDescriptions(
  category: string,
  count: number,
  priceRange: { min: number; max: number }
): Promise<ProductDescription[]> {
  
  const prompt = `Generate ${count} ${category} products in Arabic with:
  - Arabic product names
  - Detailed Arabic descriptions
  - Prices between ${priceRange.min} and ${priceRange.max} SAR
  - Relevant categories`;

  const response = await openai.chat.completions.create({
    model: "gpt-4o", // Latest model
    messages: [{ role: "user", content: prompt }],
    response_format: { type: "json_object" }
  });

  return JSON.parse(response.choices[0].message.content);
}
```

**AI Features**:
- **Product Generation**: Creates Arabic product descriptions
- **Image Generation**: Generates product images using DALL-E
- **Pricing Intelligence**: Suggests competitive pricing
- **Store Optimization**: Enhances store descriptions

### 5. Frontend Pages

#### Store Creation Wizard (`client/src/pages/create-store.tsx`)

**Purpose**: Guides users through store setup process

```typescript
export default function CreateStore() {
  const [currentStep, setCurrentStep] = useState(1);
  const [formData, setFormData] = useState<StoreWizardData>();
  
  const createStoreMutation = useMutation({
    mutationFn: (data: StoreWizardData) => api.createStore(data),
    onSuccess: (store) => {
      toast({ title: "Store created successfully!" });
      navigate(`/dashboard?store=${store.id}`);
    }
  });

  const handleSubmit = (data: StoreWizardData) => {
    createStoreMutation.mutate(data);
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-purple-50 to-pink-50">
      <StoreWizard 
        onSubmit={handleSubmit} 
        isLoading={createStoreMutation.isPending} 
      />
    </div>
  );
}
```

**Key Features**:
- **Multi-step Process**: Guided store creation
- **Template Selection**: Pre-built store templates
- **Real-time Validation**: Form validation with visual feedback
- **Arabic Support**: RTL layout and Arabic fonts

#### Dashboard (`client/src/pages/dashboard.tsx`)

**Purpose**: Store management interface with analytics

```typescript
export default function Dashboard() {
  const [selectedStore, setSelectedStore] = useState<Store | null>(null);
  
  const { data: stores } = useQuery({
    queryKey: ['/api/stores'],
    enabled: true
  });

  const { data: products } = useQuery({
    queryKey: ['/api/products', selectedStore?.id],
    enabled: !!selectedStore
  });

  const generateProductsMutation = useMutation({
    mutationFn: (data: AIProductGeneration) => 
      api.generateProducts(selectedStore!.id, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['/api/products'] });
      toast({ title: "Products generated successfully!" });
    }
  });

  return (
    <div className="min-h-screen bg-gray-50">
      <div className="flex">
        <StoreSidebar stores={stores} onStoreSelect={setSelectedStore} />
        <main className="flex-1 p-6">
          <StoreStats store={selectedStore} />
          <ProductManagement 
            products={products} 
            onGenerateProducts={generateProductsMutation.mutate}
          />
        </main>
      </div>
    </div>
  );
}
```

**Features**:
- **Store Selection**: Multi-store management
- **Analytics Dashboard**: Revenue, orders, and product metrics
- **Product Management**: CRUD operations with AI assistance
- **Real-time Updates**: Live data with React Query

#### Customer Storefront (`client/src/pages/store-view.tsx`)

**Purpose**: Public-facing store for customers

```typescript
export default function StoreView() {
  const { id } = useParams();
  const [cart, setCart] = useState<CartItem[]>([]);
  const [searchTerm, setSearchTerm] = useState("");

  const { data: store } = useQuery({
    queryKey: ['/api/stores', id],
    enabled: !!id
  });

  const { data: products } = useQuery({
    queryKey: ['/api/products', id],
    enabled: !!id
  });

  const filteredProducts = products?.filter(product =>
    product.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
    product.description.toLowerCase().includes(searchTerm.toLowerCase())
  );

  const addToCart = (product: Product) => {
    setCart(prev => {
      const existingItem = prev.find(item => item.product.id === product.id);
      if (existingItem) {
        return prev.map(item =>
          item.product.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [...prev, { product, quantity: 1 }];
    });
  };

  return (
    <div className="min-h-screen bg-white" dir="rtl">
      <StoreHeader store={store} />
      <ProductSearch onSearch={setSearchTerm} />
      <ProductGrid products={filteredProducts} onAddToCart={addToCart} />
      <ShoppingCart cart={cart} />
    </div>
  );
}
```

**Customer Features**:
- **Product Browsing**: Search and filter products
- **Shopping Cart**: Add/remove items with quantity management
- **Arabic Interface**: Full RTL support
- **Order Placement**: Complete checkout process

### 6. UI Components

#### Reusable Components (`client/src/components/`)

**Store Templates (`components/store-templates.tsx`)**:
```typescript
const templates = [
  {
    id: 'fashion',
    name: 'أزياء ومجوهرات',
    category: 'fashion',
    icon: <Shirt className="w-6 h-6" />,
    color: 'text-pink-600',
    bgColor: 'bg-pink-50',
    features: ['معرض صور متقدم', 'مقاسات متعددة', 'خصومات موسمية']
  },
  // ... more templates
];

export default function StoreTemplates({ onTemplateSelect }) {
  return (
    <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      {templates.map(template => (
        <Card key={template.id} className="cursor-pointer hover:shadow-lg">
          <CardContent className="p-6">
            <div className={`${template.bgColor} ${template.color} p-3 rounded-lg w-fit`}>
              {template.icon}
            </div>
            <h3 className="font-semibold mt-4">{template.name}</h3>
            <ul className="mt-2 space-y-1">
              {template.features.map(feature => (
                <li key={feature} className="text-sm text-gray-600">
                  ✓ {feature}
                </li>
              ))}
            </ul>
          </CardContent>
        </Card>
      ))}
    </div>
  );
}
```

**Product Form (`components/product-form.tsx`)**:
```typescript
export default function ProductForm({ storeId, onSuccess }) {
  const form = useForm<ProductFormData>({
    resolver: zodResolver(productFormSchema),
    defaultValues: {
      name: "",
      description: "",
      price: 0,
      category: "",
      stock: 0
    }
  });

  const createProductMutation = useMutation({
    mutationFn: (data: ProductFormData) => api.createProduct(storeId, data),
    onSuccess: () => {
      form.reset();
      onSuccess?.();
      toast({ title: "Product created successfully!" });
    }
  });

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(createProductMutation.mutate)}>
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>اسم المنتج</FormLabel>
              <FormControl>
                <Input placeholder="أدخل اسم المنتج" {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        {/* More form fields... */}
        <Button type="submit" disabled={createProductMutation.isPending}>
          {createProductMutation.isPending ? "جاري الإنشاء..." : "إنشاء المنتج"}
        </Button>
      </form>
    </Form>
  );
}
```

## Data Flow Architecture

### 1. Store Creation Flow
```
User Input → Form Validation → API Call → Database Insert → Success Response → Redirect to Dashboard
```

### 2. AI Product Generation Flow
```
User Request → OpenAI API Call → Parse JSON Response → Database Batch Insert → Cache Invalidation → UI Update
```

### 3. Customer Purchase Flow
```
Browse Products → Add to Cart → Checkout Form → Order Creation → Email Notification → Order Confirmation
```

## Performance Optimizations

### 1. React Query Caching
```typescript
// Hierarchical cache keys for efficient invalidation
const queryKeys = {
  stores: ['/api/stores'],
  store: (id: number) => ['/api/stores', id],
  products: (storeId: number) => ['/api/products', storeId],
  orders: (storeId: number) => ['/api/orders', storeId]
};
```

### 2. Database Indexing
```sql
-- Optimized indexes for common queries
CREATE INDEX idx_products_store_active ON products(store_id, is_active);
CREATE INDEX idx_orders_store_date ON orders(store_id, created_at);
CREATE INDEX idx_stores_domain ON stores(custom_domain);
```

### 3. Component Optimization
```typescript
// Memoized components prevent unnecessary re-renders
const ProductCard = memo(({ product, onAddToCart }) => {
  return (
    <Card className="hover:shadow-lg transition-shadow">
      <CardContent>
        <img src={product.imageUrl} alt={product.name} className="w-full h-48 object-cover" />
        <h3 className="font-semibold mt-2">{product.name}</h3>
        <p className="text-gray-600 text-sm">{product.description}</p>
        <div className="flex justify-between items-center mt-4">
          <span className="font-bold text-lg">{product.price} ر.س</span>
          <Button onClick={() => onAddToCart(product)}>
            إضافة للسلة
          </Button>
        </div>
      </CardContent>
    </Card>
  );
});
```

## Error Handling Strategy

### 1. API Error Handling
```typescript
// Centralized error handling in API client
export const apiRequest = async (endpoint: string, options?: RequestInit) => {
  try {
    const response = await fetch(endpoint, options);
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}: ${response.statusText}`);
    }
    
    return await response.json();
  } catch (error) {
    console.error('API Error:', error);
    throw error;
  }
};
```

### 2. React Error Boundaries
```typescript
class ErrorBoundary extends Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="text-center p-8">
          <h2 className="text-xl font-semibold mb-4">حدث خطأ غير متوقع</h2>
          <Button onClick={() => window.location.reload()}>
            إعادة تحميل الصفحة
          </Button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

## Security Implementation

### 1. Input Validation
```typescript
// Zod schemas prevent injection attacks
export const productFormSchema = z.object({
  name: z.string().min(1, "اسم المنتج مطلوب").max(255),
  description: z.string().min(10, "الوصف قصير جداً").max(1000),
  price: z.number().min(0.01, "السعر يجب أن يكون أكبر من صفر"),
  category: z.string().min(1, "الفئة مطلوبة"),
  stock: z.number().int().min(0, "الكمية لا يمكن أن تكون سالبة")
});
```

### 2. Environment Variables
```typescript
// Secure API key handling
const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
  dangerouslyAllowBrowser: false // Server-side only
});
```

This architecture provides a solid foundation for building scalable, maintainable e-commerce applications with AI integration and Arabic language support.