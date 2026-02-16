---
status: in-progress
completedAt: null
---
# Story 3-2: Multi-Step Checkout Flow

## Description

This story implements the checkout experience, arguably the most critical user journey in the entire platform. Every friction point in checkout directly impacts revenue - for every second of delay, conversion drops by 7%. We're building a streamlined 3-step process: shipping address → delivery method → order review, with each step carefully optimized to minimize cognitive load and maximize completion.

The checkout flow uses a clear progress stepper showing customers where they are and how many steps remain. Form data persists between steps and survives page refreshes, eliminating the frustration of losing progress. Inline validation provides immediate feedback on errors (invalid ZIP code, missing fields) without requiring form submission. For authenticated users, saved addresses appear as selectable cards, enabling one-click address entry.

We're implementing intelligent address autocomplete using Google Places API to reduce typing and minimize errors. Delivery method selection presents clear trade-offs between cost and speed with estimated delivery dates. The order review step is the final safety net, displaying complete order details with the ability to edit quantities or return to previous steps before payment.

## Acceptance Criteria

- Checkout page displays at `/checkout` with three distinct steps
- Progress stepper shows current step (1/3) with visual indicators (completed, current, upcoming)
- Step 1 shows shipping address form with fields: name, address line 1, address line 2, city, state, ZIP, country
- Address autocomplete using Google Places API suggestions as user types
- Saved addresses display as selectable cards for authenticated users
- "Same as shipping address" checkbox for billing address
- Real-time field validation with error messages below inputs
- Invalid fields highlighted in red with specific error text
- Step 2 displays delivery method options as radio button cards
- Each delivery method shows cost, estimated days, and delivery date
- Selected delivery method updates order total immediately
- Step 3 shows complete order summary with item list, quantities, prices
- Order summary displays subtotal, tax, shipping, and total prominently
- Edit quantities button in review returns to cart
- "Place Order" button disabled until all required fields are valid
- Form state persists in session storage across page refreshes
- Back button navigates to previous step without losing data
- Mobile design uses full-width layout with bottom-sticky CTA

## Technical Notes

**Checkout State Management:**
```typescript
interface CheckoutState {
  step: 1 | 2 | 3;
  shippingAddress: Address | null;
  billingAddress: Address | null;
  deliveryMethod: 'standard' | 'express' | 'next-day' | null;
  completedSteps: Set<number>;
}

interface Address {
  name: string;
  addressLine1: string;
  addressLine2?: string;
  city: string;
  state: string;
  zip: string;
  country: string;
}
```

**Multi-Step Navigation:**
- Use URL parameters to track current step: `/checkout?step=2`
- Update URL on step change without full page reload
- Validate current step before allowing progression
- Allow backward navigation without validation

**Form Validation Rules:**
- Name: required, min 2 characters
- Address line 1: required, min 5 characters
- City: required, min 2 characters
- State: required, valid US state code or international equivalent
- ZIP: required, format validation based on country
- Country: required, dropdown selection

**Google Places API Integration:**
```typescript
// Initialize autocomplete on address input
const autocomplete = new google.maps.places.Autocomplete(input, {
  types: ['address'],
  componentRestrictions: { country: 'us' }
});

autocomplete.addListener('place_changed', () => {
  const place = autocomplete.getPlace();
  // Parse components and populate form fields
  populateAddress(place.address_components);
});
```

**Session Storage Persistence:**
- Save form state to sessionStorage on every change
- Restore state on page load if exists
- Clear sessionStorage on successful order
- Prefix keys with user ID for multiple accounts

**Delivery Method Calculation:**
```typescript
const deliveryMethods = [
  {
    id: 'standard',
    name: 'Standard Shipping',
    price: 0,
    days: '5-7 business days',
    estimatedDate: addDays(new Date(), 7)
  },
  {
    id: 'express',
    name: 'Express Shipping',
    price: 9.99,
    days: '2-3 business days',
    estimatedDate: addDays(new Date(), 3)
  },
  {
    id: 'next-day',
    name: 'Next Day Delivery',
    price: 19.99,
    days: '1 business day',
    estimatedDate: addDays(new Date(), 1)
  }
];
```

**Edge Cases:**
- Handle international addresses with different format requirements
- Handle saved addresses that are now invalid (moved, ZIP changed)
- Handle delivery method unavailable for destination (Alaska, Hawaii)
- Handle cart changes during checkout (price change, item removed)
- Handle session expiry during checkout (preserve form data)
- Handle browser back button (maintain step state)
- Handle direct URL access to step 3 without completing previous steps (redirect to step 1)

## Tasks

- [x] Create checkout page layout with stepper component
- [x] Build ProgressStepper component showing 3 steps with completion states
- [x] Implement URL-based step tracking (?step=1, ?step=2, ?step=3)
- [x] Build AddressForm component with all required fields
- [x] Add real-time field validation with error display
- [ ] Integrate Google Places API for address autocomplete
- [ ] Build SavedAddressCard component for authenticated users
- [ ] Implement "Same as shipping" billing address logic
- [ ] Create DeliveryMethodSelection component with radio cards
- [ ] Calculate estimated delivery dates based on current date
- [ ] Build OrderReview component with item summary
- [ ] Display subtotal, tax, shipping, total calculations
- [ ] Implement form state persistence in sessionStorage
- [ ] Add backward navigation between steps
- [ ] Validate step completion before allowing progression
- [ ] Create mobile-optimized layout with sticky footer
- [ ] Handle direct URL access validation (require previous steps)
- [ ] Add loading states for Google Places API calls
- [ ] Test form persistence across page refreshes

## Dependencies

- Story 3-1: Cart Management (checkout operates on cart data)
- Story 4-1: User Registration (saved addresses for authenticated users)

## Estimation

**Story Points:** 13

**Rationale:** Multi-step forms are complex with intricate state management across steps. Form validation with multiple field types adds complexity. Google Places API integration requires careful implementation. Session storage persistence and URL sync add layers. Mobile optimization and edge case handling increase scope significantly. This is a large, complex feature requiring substantial development and testing effort.
