---
status: in-progress
completedAt: null
---
# Story 3-3: Payment Processing Integration

## Description

This story integrates Stripe as our payment processor, handling one of the most sensitive and critical aspects of the e-commerce platform. Payment processing must be both secure (PCI DSS compliant) and frictionless - 28% of cart abandonment is due to payment issues. We're implementing Stripe Elements for secure card data collection that never touches our servers, maintaining PCI compliance without the burden of handling sensitive payment information.

Beyond traditional card payments, we're integrating Stripe's Payment Request API to enable Apple Pay and Google Pay, which convert 1.5-2x better than manual card entry on mobile devices. These express payment methods allow customers to complete checkout in seconds using saved payment information from their device or browser. Stripe's payment intents API handles the complex payment lifecycle including authorization, capture, and webhooks for asynchronous confirmation.

Error handling is critical - we're implementing user-friendly error messages for all payment failure scenarios (declined card, insufficient funds, expired card) with actionable guidance. The payment confirmation flow includes detailed order summaries and email confirmation. All payment communication happens over HTTPS with additional security measures including CSRF tokens and rate limiting on payment endpoints.

## Acceptance Criteria

- Stripe SDK initialized with publishable key from environment variables
- Payment intent created on server when customer proceeds to payment step
- Stripe Elements renders secure card input fields (number, expiry, CVC)
- Card field validation provides real-time feedback (invalid card number, expired date)
- Apple Pay button appears on supported devices (iPhone, iPad, Safari)
- Google Pay button appears on supported browsers (Chrome, Android)
- Express payment methods auto-fill shipping address from wallet
- "Place Order" button initiates payment confirmation flow
- Payment errors display user-friendly messages (card declined, network error)
- Successful payment redirects to order confirmation page
- Failed payment allows retry without losing form data
- Payment processing shows loading state with spinner overlay
- Payment intent amount matches order total exactly
- Currency is set based on store location (USD for v1)
- PCI compliance maintained (no card data stored on server)
- Rate limiting prevents brute-force payment attempts (max 5/hour per IP)

## Technical Notes

**Stripe SDK Setup:**
```typescript
import { loadStripe } from '@stripe/stripe-js';
import { Elements, CardElement, PaymentRequestButtonElement } from '@stripe/react-stripe-js';

const stripePromise = loadStripe(process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY);
```

**Payment Intent Creation:**
```typescript
// Server-side API route
export async function POST(req: Request) {
  const { amount, currency } = await req.json();

  const paymentIntent = await stripe.paymentIntents.create({
    amount: Math.round(amount * 100), // Convert to cents
    currency: currency || 'usd',
    automatic_payment_methods: { enabled: true },
    metadata: {
      orderId: orderId,
      userId: userId
    }
  });

  return Response.json({ clientSecret: paymentIntent.client_secret });
}
```

**Card Element Configuration:**
```typescript
const cardElementOptions = {
  style: {
    base: {
      fontSize: '16px',
      color: '#1f2937',
      '::placeholder': { color: '#9ca3af' }
    },
    invalid: { color: '#ef4444' }
  },
  hidePostalCode: true // Collected in shipping address
};
```

**Payment Confirmation:**
```typescript
const handleSubmit = async (e: React.FormEvent) => {
  e.preventDefault();

  const { error, paymentIntent } = await stripe.confirmCardPayment(
    clientSecret,
    {
      payment_method: {
        card: elements.getElement(CardElement),
        billing_details: {
          name: billingName,
          address: billingAddress
        }
      }
    }
  );

  if (error) {
    setError(getErrorMessage(error.code));
  } else if (paymentIntent.status === 'succeeded') {
    // Create order record and redirect to confirmation
    await createOrder(paymentIntent.id);
    router.push(`/order-confirmation?orderId=${orderId}`);
  }
};
```

**Apple Pay / Google Pay:**
```typescript
const paymentRequest = stripe.paymentRequest({
  country: 'US',
  currency: 'usd',
  total: {
    label: 'ShopFlow Order',
    amount: Math.round(total * 100)
  },
  requestPayerName: true,
  requestPayerEmail: true,
  requestShipping: true,
  shippingOptions: deliveryMethods.map(m => ({
    id: m.id,
    label: m.name,
    detail: m.days,
    amount: Math.round(m.price * 100)
  }))
});
```

**Error Message Mapping:**
```typescript
function getErrorMessage(code: string): string {
  const messages = {
    card_declined: 'Your card was declined. Please try a different payment method.',
    insufficient_funds: 'Your card has insufficient funds.',
    expired_card: 'Your card has expired. Please use a different card.',
    incorrect_cvc: 'The CVC code is incorrect.',
    processing_error: 'An error occurred while processing your payment. Please try again.',
    // ... more error codes
  };
  return messages[code] || 'An unexpected error occurred. Please try again.';
}
```

**Security Measures:**
- HTTPS only for all payment pages (enforced in middleware)
- CSRF tokens on payment submission
- Rate limiting: max 5 payment attempts per hour per IP
- Stripe webhook signature verification for server events
- Environment variable validation on startup

**Edge Cases:**
- Handle payment that succeeds but order creation fails (webhook fallback)
- Handle 3D Secure authentication popup blockers
- Handle payment timeout after 10 minutes (expire payment intent)
- Handle currency mismatch (order in EUR, payment in USD)
- Handle partial payments for split orders
- Handle payment method requiring additional action (3DS2, SCA)
- Handle network errors during payment confirmation (retry logic)

## Tasks

- [x] Set up Stripe SDK and install required packages
- [x] Configure Stripe API keys in environment variables
- [x] Create payment intent API endpoint on server
- [ ] Build PaymentForm component with Stripe Elements
- [ ] Integrate CardElement for secure card input
- [ ] Add real-time card validation and error display
- [ ] Implement payment confirmation flow
- [ ] Create error message mapping for all Stripe error codes
- [ ] Integrate Apple Pay using Payment Request Button
- [ ] Integrate Google Pay using Payment Request Button
- [ ] Build payment loading state with overlay spinner
- [ ] Implement payment retry logic on failure
- [ ] Add payment attempt rate limiting (5/hour per IP)
- [ ] Create Stripe webhook endpoint for payment confirmation
- [ ] Verify webhook signatures for security
- [ ] Build order creation logic after successful payment
- [ ] Test 3D Secure authentication flows
- [ ] Test payment failures and error handling
- [ ] Add CSRF protection on payment submission

## Dependencies

- Story 3-2: Checkout Flow (payment is final step of checkout)
- Story 3-4: Order Confirmation (redirect after successful payment)

## Estimation

**Story Points:** 13

**Rationale:** Payment integration is high-stakes and complex, requiring careful security implementation. Stripe Elements integration is well-documented but requires attention to detail. Apple Pay and Google Pay add complexity with device/browser compatibility. Error handling for all payment scenarios is extensive. Webhook setup and 3D Secure authentication increase scope. This is a large feature with significant security implications requiring thorough testing.
