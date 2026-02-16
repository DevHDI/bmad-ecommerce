---
status: backlog
completedAt: null
---
# Story 5-1: Product Administration Panel

## Description

This story implements the admin interface for product management, enabling store owners to create, edit, and manage their product catalog efficiently. Product management is the most frequently used admin function, accessed daily by 89% of e-commerce admins. We're building a comprehensive product administration system with powerful search, bulk actions, and an intuitive editing interface that handles complex product data including images, variants, and inventory.

The product list uses a data table component with server-side pagination, sorting, and filtering to handle catalogs of thousands of products without performance degradation. Quick actions allow inline editing of common fields (price, stock) while a full edit modal provides access to all product attributes. Image management includes drag-and-drop upload with automatic resizing, cropping, and optimization for web delivery.

Bulk actions enable efficient management at scale - admins can select multiple products and perform batch operations like publish/unpublish, category assignment, or deletion. Rich text editing for product descriptions uses a WYSIWYG editor that generates clean HTML. The form includes comprehensive validation to prevent invalid product data from entering the system.

## Acceptance Criteria

- Admin product list page displays at `/admin/products` (admin role required)
- Data table shows products with columns: image, name, SKU, price, stock, category, status
- Table supports sorting by clicking column headers (name, price, stock, date)
- Search bar filters products by name, SKU, or description
- Category filter dropdown narrows results to specific category
- Status filter shows active, draft, or archived products
- Server-side pagination displays 25 products per page with page numbers
- "Create Product" button opens product creation form modal
- Product form includes fields: name, slug, description, price, compare-at price, SKU
- Rich text editor (Tiptap) for product description with formatting options
- Category selector with hierarchical dropdown
- Image upload area supports drag-and-drop and file picker
- Multiple images can be uploaded, first image is primary
- Image preview shows all uploaded images with reorder via drag-and-drop
- Delete image button removes image with confirmation
- Stock management fields: quantity, low stock threshold
- Status dropdown: active, draft, archived
- Form validation prevents submission with missing required fields
- "Save" button creates/updates product and shows success toast
- Row actions menu includes Edit, Duplicate, Delete options
- Checkbox in each row allows multi-select for bulk actions
- Bulk actions toolbar appears when products selected
- Bulk actions: Publish, Unpublish, Delete (with confirmation)
- "Select All" checkbox selects all products on current page

## Technical Notes

**Data Table Implementation:**
```typescript
interface ProductTableProps {
  products: Product[];
  totalCount: number;
  currentPage: number;
  pageSize: number;
  sortBy: string;
  sortOrder: 'asc' | 'desc';
  onSort: (column: string) => void;
  onPageChange: (page: number) => void;
}

// Server-side query
async function getProducts(filters: ProductFilters) {
  const { search, category, status, page, pageSize, sortBy, sortOrder } = filters;

  return prisma.product.findMany({
    where: {
      AND: [
        search ? {
          OR: [
            { name: { contains: search, mode: 'insensitive' } },
            { sku: { contains: search, mode: 'insensitive' } },
            { description: { contains: search, mode: 'insensitive' } },
          ],
        } : {},
        category ? { categoryId: category } : {},
        status ? { status } : {},
      ],
    },
    orderBy: { [sortBy]: sortOrder },
    skip: (page - 1) * pageSize,
    take: pageSize,
    include: { category: true },
  });
}
```

**Rich Text Editor (Tiptap):**
```typescript
import { useEditor, EditorContent } from '@tiptap/react';
import StarterKit from '@tiptap/starter-kit';

const editor = useEditor({
  extensions: [StarterKit],
  content: initialDescription,
  editorProps: {
    attributes: {
      class: 'prose prose-sm max-w-none',
    },
  },
});

// Get HTML output
const description = editor.getHTML();
```

**Image Upload with S3:**
```typescript
async function uploadProductImages(files: File[], productId: string) {
  const uploadedUrls = [];

  for (const file of files) {
    // Resize image to max 2000x2000 for web
    const resized = await resizeImage(file, 2000, 2000);

    const key = `products/${productId}/${Date.now()}-${file.name}`;
    await s3.upload({
      Bucket: process.env.S3_BUCKET,
      Key: key,
      Body: resized,
      ContentType: file.type,
    });

    uploadedUrls.push(`https://${process.env.S3_BUCKET}.s3.amazonaws.com/${key}`);
  }

  return uploadedUrls;
}
```

**Bulk Actions:**
```typescript
async function bulkUpdateProducts(productIds: string[], action: BulkAction) {
  switch (action) {
    case 'publish':
      await prisma.product.updateMany({
        where: { id: { in: productIds } },
        data: { status: 'active' },
      });
      break;
    case 'unpublish':
      await prisma.product.updateMany({
        where: { id: { in: productIds } },
        data: { status: 'draft' },
      });
      break;
    case 'delete':
      await prisma.product.deleteMany({
        where: { id: { in: productIds } },
      });
      break;
  }
}
```

**Slug Auto-Generation:**
```typescript
function generateSlug(name: string): string {
  return name
    .toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/^-+|-+$/g, '');
}

// Check for uniqueness
async function ensureUniqueSlug(slug: string, productId?: string): Promise<string> {
  let finalSlug = slug;
  let counter = 1;

  while (true) {
    const existing = await prisma.product.findUnique({
      where: { slug: finalSlug },
    });

    if (!existing || existing.id === productId) {
      return finalSlug;
    }

    finalSlug = `${slug}-${counter}`;
    counter++;
  }
}
```

**Edge Cases:**
- Handle duplicate SKUs (validate uniqueness before save)
- Handle very large images (resize before upload, show progress)
- Handle slug conflicts (auto-append number for uniqueness)
- Handle missing categories (prevent orphaned products)
- Handle bulk delete of in-stock products (show warning)
- Handle network errors during image upload (retry logic)
- Handle deleting products with active orders (soft delete, hide from catalog)
- Handle concurrent edits by multiple admins (optimistic locking)

## Tasks

- [ ] Create admin layout with sidebar navigation
- [ ] Build product list page at `/admin/products`
- [ ] Implement data table component with sortable columns
- [ ] Add server-side pagination logic
- [ ] Build search functionality filtering by name/SKU/description
- [ ] Add category and status filter dropdowns
- [ ] Create product form modal with all fields
- [ ] Integrate Tiptap rich text editor for description
- [ ] Build image upload component with drag-and-drop
- [ ] Implement image upload to S3 with resizing
- [ ] Add image reordering with drag-and-drop
- [ ] Build category selector with hierarchical dropdown
- [ ] Add form validation with Zod schema
- [ ] Implement slug auto-generation from product name
- [ ] Create product creation API endpoint
- [ ] Create product update API endpoint
- [ ] Add row actions menu (Edit, Duplicate, Delete)
- [ ] Implement multi-select checkboxes
- [ ] Build bulk actions toolbar
- [ ] Add bulk publish/unpublish/delete functionality
- [ ] Test with large product catalog (1000+ products)

## Dependencies

- Story 1-3: Development Environment (requires database schema)
- Story 4-1: User Registration (admin role enforcement)
- Story 2-2: Category System (products assigned to categories)

## Estimation

**Story Points:** 13

**Rationale:** Admin product management is a large feature with complex UI requirements. Data table with sorting, filtering, and pagination adds substantial complexity. Rich text editor integration and image upload with S3 require careful implementation. Bulk actions and multi-select UI increase scope. Form validation and edge case handling (slug conflicts, duplicate SKUs) add complexity. Overall this is a large, complex feature requiring significant development effort.
