---
name: web-builder
description: Build modern web pages and components using SuperDesign patterns with ShadCN UI components, never using basic HTML/CSS when design system components are available
license: Apache-2.0
allowed-tools:
  - Bash(npx shadcn*:*)
  - SlashCommand(/superdesign*)
  - Read
  - Write
  - Edit
  - Glob
  - Grep
metadata:
  version: "1.0.0"
  design-system: "SuperDesign + ShadCN"
---

# Web Builder

Expert web development using SuperDesign patterns with ShadCN UI components for modern, polished interfaces.

## Core Principles

### MANDATORY STANDARDS
- **ALWAYS** use SuperDesign and ShadCN for web page creation
- **DEFAULT** UI framework: ShadCN components with SuperDesign styling
- **APPLIES TO**: All web projects (Next.js, React, or any framework)
- **NO EXCEPTIONS**: Don't use basic HTML/CSS when ShadCN components are available
- **COMPONENT PREFERENCE**: Use ShadCN components over custom components when possible
- **STYLING**: Integrate SuperDesign patterns and principles

## Design System Workflow

### Step 1: Component Discovery
**ALWAYS check for appropriate ShadCN components first**

```bash
# Search for available components
npx shadcn@latest add --help

# Common components
npx shadcn@latest add button
npx shadcn@latest add input
npx shadcn@latest add form
npx shadcn@latest add card
npx shadcn@latest add dialog
npx shadcn@latest add dropdown-menu
npx shadcn@latest add tabs
npx shadcn@latest add table
npx shadcn@latest add toast
npx shadcn@latest add alert
```

### Step 2: SuperDesign Integration
**Apply SuperDesign principles for layouts and styling**

- Use `/superdesign` command for generating polished UIs
- Follow SuperDesign composition patterns
- Implement SuperDesign spacing and typography
- Use SuperDesign color systems
- Apply SuperDesign motion principles

### Step 3: Implementation
**Use latest versions of both systems**

```typescript
// Example: Button with ShadCN + SuperDesign patterns
import { Button } from '@/components/ui/button'

export function CTAButton() {
  return (
    <Button
      size="lg"
      className="bg-gradient-to-r from-blue-600 to-blue-800
                 hover:from-blue-700 hover:to-blue-900
                 transition-all duration-200 ease-in-out
                 shadow-lg hover:shadow-xl"
    >
      Get Started
    </Button>
  )
}
```

## ShadCN Component Library

### Form Components
```typescript
// Form with validation
import { Button } from '@/components/ui/button'
import { Form, FormControl, FormField, FormItem, FormLabel, FormMessage } from '@/components/ui/form'
import { Input } from '@/components/ui/input'

// Select
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'

// Checkbox
import { Checkbox } from '@/components/ui/checkbox'

// Radio Group
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group'

// Switch
import { Switch } from '@/components/ui/switch'

// Textarea
import { Textarea } from '@/components/ui/textarea'
```

### Layout Components
```typescript
// Card
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from '@/components/ui/card'

// Tabs
import { Tabs, TabsContent, TabsList, TabsTrigger } from '@/components/ui/tabs'

// Separator
import { Separator } from '@/components/ui/separator'

// Accordion
import { Accordion, AccordionContent, AccordionItem, AccordionTrigger } from '@/components/ui/accordion'
```

### Navigation Components
```typescript
// Navigation Menu
import { NavigationMenu, NavigationMenuContent, NavigationMenuItem, NavigationMenuLink, NavigationMenuList, NavigationMenuTrigger } from '@/components/ui/navigation-menu'

// Breadcrumb
import { Breadcrumb, BreadcrumbItem, BreadcrumbLink, BreadcrumbList, BreadcrumbPage, BreadcrumbSeparator } from '@/components/ui/breadcrumb'

// Pagination
import { Pagination, PaginationContent, PaginationItem, PaginationLink, PaginationNext, PaginationPrevious } from '@/components/ui/pagination'
```

### Feedback Components
```typescript
// Alert
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'

// Toast
import { useToast } from '@/components/ui/use-toast'
import { toast } from '@/components/ui/use-toast'

// Dialog
import { Dialog, DialogContent, DialogDescription, DialogHeader, DialogTitle, DialogTrigger } from '@/components/ui/dialog'

// Alert Dialog
import { AlertDialog, AlertDialogAction, AlertDialogCancel, AlertDialogContent, AlertDialogDescription, AlertDialogFooter, AlertDialogHeader, AlertDialogTitle, AlertDialogTrigger } from '@/components/ui/alert-dialog'

// Progress
import { Progress } from '@/components/ui/progress'

// Skeleton
import { Skeleton } from '@/components/ui/skeleton'
```

### Data Display Components
```typescript
// Table
import { Table, TableBody, TableCell, TableHead, TableHeader, TableRow } from '@/components/ui/table'

// Badge
import { Badge } from '@/components/ui/badge'

// Avatar
import { Avatar, AvatarFallback, AvatarImage } from '@/components/ui/avatar'
```

### Overlay Components
```typescript
// Popover
import { Popover, PopoverContent, PopoverTrigger } from '@/components/ui/popover'

// Dropdown Menu
import { DropdownMenu, DropdownMenuContent, DropdownMenuItem, DropdownMenuLabel, DropdownMenuSeparator, DropdownMenuTrigger } from '@/components/ui/dropdown-menu'

// Sheet (Drawer)
import { Sheet, SheetContent, SheetDescription, SheetHeader, SheetTitle, SheetTrigger } from '@/components/ui/sheet'

// Tooltip
import { Tooltip, TooltipContent, TooltipProvider, TooltipTrigger } from '@/components/ui/tooltip'
```

## SuperDesign Patterns

### Layout Patterns
```typescript
// Hero Section
<section className="relative overflow-hidden bg-gradient-to-b from-blue-50 to-white dark:from-gray-900 dark:to-gray-800">
  <div className="container mx-auto px-4 py-24 sm:px-6 lg:px-8">
    <div className="grid gap-12 lg:grid-cols-2 lg:gap-8">
      {/* Content */}
    </div>
  </div>
</section>

// Feature Grid
<div className="grid gap-8 md:grid-cols-2 lg:grid-cols-3">
  {features.map((feature) => (
    <Card key={feature.id}>
      <CardHeader>
        <CardTitle>{feature.title}</CardTitle>
        <CardDescription>{feature.description}</CardDescription>
      </CardHeader>
    </Card>
  ))}
</div>

// Bento Grid
<div className="grid auto-rows-[192px] grid-cols-3 gap-4">
  <div className="col-span-2 row-span-2">{/* Large item */}</div>
  <div className="col-span-1 row-span-1">{/* Small item */}</div>
  <div className="col-span-1 row-span-1">{/* Small item */}</div>
</div>
```

### Color Systems
```typescript
// Semantic Colors (SuperDesign)
const colors = {
  primary: 'bg-blue-600 hover:bg-blue-700',
  secondary: 'bg-gray-600 hover:bg-gray-700',
  success: 'bg-green-600 hover:bg-green-700',
  danger: 'bg-red-600 hover:bg-red-700',
  warning: 'bg-yellow-600 hover:bg-yellow-700',
}

// Gradients
const gradients = {
  sunset: 'bg-gradient-to-r from-orange-500 via-pink-500 to-purple-600',
  ocean: 'bg-gradient-to-r from-blue-600 to-cyan-500',
  forest: 'bg-gradient-to-r from-green-600 to-emerald-500',
  midnight: 'bg-gradient-to-r from-purple-900 via-blue-900 to-indigo-900',
}
```

### Typography Patterns
```typescript
// Headings
<h1 className="text-4xl font-bold tracking-tight sm:text-6xl lg:text-7xl">
  Main Heading
</h1>

<h2 className="text-3xl font-semibold tracking-tight sm:text-4xl lg:text-5xl">
  Section Heading
</h2>

<h3 className="text-2xl font-semibold tracking-tight">
  Subsection Heading
</h3>

// Body Text
<p className="text-lg leading-relaxed text-gray-600 dark:text-gray-300">
  Body text with proper line height and color
</p>

// Small Text
<p className="text-sm text-gray-500 dark:text-gray-400">
  Secondary information
</p>
```

### Motion & Transitions
```typescript
// Smooth Transitions
className="transition-all duration-200 ease-in-out"

// Hover Effects
className="hover:scale-105 hover:shadow-xl transition-transform duration-200"

// Focus States
className="focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"

// Loading States
className="animate-pulse"
className="animate-spin"
```

## Component Composition Pattern

```typescript
// Example: Feature Card with SuperDesign + ShadCN
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'

interface FeatureCardProps {
  title: string
  description: string
  icon: React.ReactNode
  badge?: string
}

export function FeatureCard({ title, description, icon, badge }: FeatureCardProps) {
  return (
    <Card className="group relative overflow-hidden transition-all duration-200 hover:shadow-lg">
      <CardHeader>
        <div className="flex items-center justify-between">
          <div className="rounded-lg bg-blue-50 p-3 dark:bg-blue-900/20">
            {icon}
          </div>
          {badge && (
            <Badge variant="secondary" className="text-xs">
              {badge}
            </Badge>
          )}
        </div>
        <CardTitle className="mt-4">{title}</CardTitle>
        <CardDescription className="text-base">{description}</CardDescription>
      </CardHeader>
      <CardContent>
        <Button variant="ghost" className="group-hover:translate-x-1 transition-transform">
          Learn more →
        </Button>
      </CardContent>
    </Card>
  )
}
```

## Setup Requirements

### Initial ShadCN Setup
```bash
# Initialize ShadCN
npx shadcn@latest init

# Follow prompts for:
# - TypeScript: Yes
# - Style: Default
# - Base color: Slate
# - Global CSS: src/app/globals.css
# - CSS variables: Yes
# - Tailwind config: tailwind.config.ts
# - Components: src/components
# - Utils: src/lib/utils
# - React Server Components: Yes
```

### Required Dependencies
```json
{
  "dependencies": {
    "class-variance-authority": "^0.7.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^2.2.0",
    "lucide-react": "^0.309.0"
  }
}
```

## Development Workflow

### 1. Identify Requirement
- Understand the UI/UX requirements
- Identify which ShadCN components fit
- Plan SuperDesign layout patterns

### 2. Component Selection
- Search ShadCN component library
- Choose appropriate components
- Install needed components

### 3. Implementation
- Start with ShadCN base components
- Apply SuperDesign styling patterns
- Add responsive design
- Implement motion/transitions

### 4. Refinement
- Check accessibility
- Test responsiveness
- Verify design consistency
- Optimize performance

## Anti-Patterns (FORBIDDEN)

### Component Anti-Patterns
- ❌ Using basic `<button>` instead of ShadCN Button
- ❌ Custom form inputs instead of ShadCN Form
- ❌ DIY modals instead of ShadCN Dialog
- ❌ Plain HTML tables instead of ShadCN Table
- ❌ Custom dropdowns instead of ShadCN DropdownMenu

### Design Anti-Patterns
- ❌ Inconsistent spacing
- ❌ Non-responsive layouts
- ❌ Missing hover/focus states
- ❌ Poor color contrast
- ❌ No loading states
- ❌ Missing error handling UI

## Accessibility Requirements

### MUST Include
- ✅ Semantic HTML structure
- ✅ ARIA labels where needed
- ✅ Keyboard navigation support
- ✅ Focus indicators
- ✅ Screen reader support
- ✅ Color contrast (WCAG AA minimum)
- ✅ Alt text for images
- ✅ Proper heading hierarchy

## Responsive Design

### Breakpoints (Tailwind)
```typescript
// Mobile first approach
sm: '640px'   // Small devices
md: '768px'   // Tablets
lg: '1024px'  // Desktops
xl: '1280px'  // Large desktops
2xl: '1536px' // Extra large desktops

// Example usage
className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4"
```

## When to Use This Skill

- Creating new web pages
- Building UI components
- Designing forms and inputs
- Implementing navigation
- Creating data tables
- Building dashboards
- Designing landing pages
- Creating modal/dialog systems
- Implementing feedback systems
- Any web UI development

## Success Criteria

Web page is complete when:
- ✅ Uses ShadCN components appropriately
- ✅ Follows SuperDesign patterns
- ✅ Fully responsive
- ✅ Accessible (WCAG AA)
- ✅ Smooth transitions/animations
- ✅ Proper loading states
- ✅ Error handling UI
- ✅ Consistent design system
