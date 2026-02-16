---
status: review
completedAt: "2026-02-08"
---
# Story 4-2: User Profile Management

## Description

This story implements user profile management features that allow customers to maintain their account information, shipping addresses, and avatar. A complete, accurate profile improves checkout conversion by 35% through saved addresses and reduces customer support inquiries about order tracking and account access. We're building an intuitive profile interface that guides users through completion with a progress indicator.

The profile page features editable fields for personal information (name, email, phone), with inline validation and save confirmation. Address management supports multiple shipping addresses stored as separate records, each with a default designation. Users can add, edit, delete, and set default addresses, streamlining the checkout process. Avatar upload includes client-side image cropping to ensure consistent sizing and aspect ratio.

Profile completion tracking encourages users to fill in all fields by showing a percentage-based progress bar. Incomplete profiles display actionable prompts ("Add a phone number", "Upload an avatar") with direct links to relevant sections. All changes are saved with optimistic UI updates and success toasts for immediate feedback.

## Acceptance Criteria

- Profile page displays at `/account/profile` for authenticated users
- Profile form shows current values for name, email, phone number
- Fields are editable with save button triggering update API call
- Real-time validation on email format and phone number format
- Profile update shows success toast and refreshes displayed data
- Address section displays list of all saved addresses
- Each address card shows full address with "Set as Default" and "Delete" buttons
- Default address is visually indicated with badge or icon
- "Add New Address" button opens form modal for new address entry
- Address form includes all fields: name, line 1, line 2, city, state, ZIP, country
- Address CRUD operations update immediately with optimistic UI
- Avatar upload button opens file picker accepting image files only (JPEG, PNG, WebP)
- Selected image displays in crop modal with adjustable crop area
- Crop area maintains 1:1 aspect ratio for circular avatar display
- Cropped image uploads to storage (S3, Cloudinary) and saves URL to user record
- Profile completion indicator shows percentage (e.g., "Profile 80% complete")
- Incomplete sections show prompts with action links
- All form submissions include loading states and error handling

## Technical Notes

**Profile Form State:**
```typescript
interface ProfileFormData {
  name: string;
  email: string;
  phone: string;
}

const profileSchema = z.object({
  name: z.string().min(2, 'Name must be at least 2 characters'),
  email: z.string().email('Invalid email format'),
  phone: z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number'),
});
```

**Address CRUD Operations:**
```typescript
// Create address
POST /api/user/addresses
Body: { name, addressLine1, addressLine2, city, state, zip, country }

// Update address
PATCH /api/user/addresses/:id
Body: { /* updated fields */ }

// Delete address
DELETE /api/user/addresses/:id

// Set default
PATCH /api/user/addresses/:id/set-default
```

**Image Crop with react-image-crop:**
```typescript
import ReactCrop, { Crop } from 'react-image-crop';

const [crop, setCrop] = useState<Crop>({
  unit: '%',
  width: 50,
  height: 50,
  x: 25,
  y: 25,
  aspect: 1, // 1:1 ratio for circular avatar
});

// After crop, generate blob and upload
const croppedBlob = await getCroppedImg(imageRef.current, crop);
const formData = new FormData();
formData.append('avatar', croppedBlob, 'avatar.jpg');

const response = await fetch('/api/user/avatar', {
  method: 'POST',
  body: formData,
});
```

**Profile Completion Calculation:**
```typescript
function calculateProfileCompletion(user: User): number {
  const fields = [
    user.name,
    user.email,
    user.phone,
    user.avatar,
    user.addresses.length > 0,
  ];

  const completed = fields.filter(Boolean).length;
  return Math.round((completed / fields.length) * 100);
}
```

**Avatar Upload to S3:**
```typescript
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

const s3 = new S3Client({ region: process.env.AWS_REGION });

export async function uploadAvatar(file: File, userId: string) {
  const key = `avatars/${userId}/${Date.now()}-${file.name}`;

  await s3.send(new PutObjectCommand({
    Bucket: process.env.S3_BUCKET,
    Key: key,
    Body: Buffer.from(await file.arrayBuffer()),
    ContentType: file.type,
  }));

  return `https://${process.env.S3_BUCKET}.s3.amazonaws.com/${key}`;
}
```

**Optimistic UI Updates:**
```typescript
const handleSave = async (data: ProfileFormData) => {
  // Update UI immediately
  setProfile(data);

  try {
    await updateProfile(data);
    toast.success('Profile updated successfully');
  } catch (error) {
    // Rollback on error
    setProfile(originalProfile);
    toast.error('Failed to update profile');
  }
};
```

**Edge Cases:**
- Handle invalid image files (show error for non-image uploads)
- Handle very large images (resize before crop, max 10MB)
- Handle network errors during save (retry logic, show error toast)
- Handle email change requiring verification (send confirmation email)
- Handle phone number format for international numbers
- Handle deleting last address (prevent if used in active order)
- Handle default address deletion (auto-set another as default)
- Handle concurrent profile updates from multiple tabs (last-write-wins)

## Tasks

- [x] Create profile page layout at `/account/profile`
- [x] Build profile form with name, email, phone fields
- [x] Add form validation with Zod schema
- [x] Implement profile update API endpoint
- [x] Add optimistic UI updates for profile save
- [x] Create success/error toast notifications
- [x] Build address list component displaying all addresses
- [x] Create AddressCard component with default badge
- [x] Implement "Add New Address" modal with form
- [x] Build address form with all required fields
- [x] Create address CRUD API endpoints
- [x] Implement set default address functionality
- [x] Add address delete with confirmation dialog
- [x] Build avatar upload component with file picker
- [x] Integrate react-image-crop for crop functionality
- [x] Implement image cropping with 1:1 aspect ratio
- [x] Create avatar upload API endpoint with S3 integration
- [x] Build profile completion indicator component
- [x] Calculate completion percentage based on filled fields
- [x] Add prompts for incomplete profile sections
- [x] Test all CRUD operations with loading states

## Dependencies

- Story 4-1: User Registration (requires authenticated user context)

## Estimation

**Story Points:** 5

**Rationale:** Profile form is straightforward with standard CRUD operations. Address management adds moderate complexity with multiple operations. Image upload with cropping requires integration of react-image-crop library and S3 setup, increasing complexity. Profile completion calculation is simple logic. Overall moderate complexity with well-established patterns.
