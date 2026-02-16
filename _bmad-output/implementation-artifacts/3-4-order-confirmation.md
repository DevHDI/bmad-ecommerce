---
status: backlog
completedAt: null
---
# Story 3-4: Order Confirmation and Email Notifications

## Description

This story implements the post-purchase experience that confirms successful orders and keeps customers informed throughout the fulfillment process. The order confirmation page is the first touchpoint after payment, reassuring customers their purchase was successful and providing essential order details. Transactional emails are critical for trust - 64% of customers expect immediate order confirmation, and shipping notifications reduce support inquiries by 40%.

We're integrating Resend as our transactional email service, providing reliable delivery with excellent developer experience. Email templates are designed with HTML/CSS for broad email client compatibility, featuring order summaries, itemized lists, shipping information, and prominent tracking links. The order confirmation email sends immediately after successful payment, while shipping notifications trigger when the admin marks orders as shipped.

Order tracking links integrate with major carriers (UPS, FedEx, USPS) and display estimated delivery dates. The confirmation page includes printable invoice functionality and easy access to customer support. All emails include unsubscribe links and preference management for compliance with email regulations (CAN-SPAM, GDPR).

## Acceptance Criteria

- Order confirmation page displays at `/order-confirmation?orderId=[id]` after successful payment
- Page shows order number, date, status, and estimated delivery
- Complete item list with images, names, quantities, and prices
- Shipping address and billing address displayed
- Payment method shown (last 4 digits of card)
- Order total breakdown (subtotal, tax, shipping, total)
- Print invoice button generates printer-friendly format
- Confirmation email sends within 30 seconds of order placement
- Email includes all order details matching confirmation page
- Shipping notification email sends when order status changes to "shipped"
- Shipping email includes tracking number and carrier link
- Tracking link opens carrier website with package tracking
- Emails are mobile-responsive and render correctly in all major clients
- Email preference center allows opting out of marketing (not transactional)
- All emails include company contact information and support link

## Technical Notes

**Resend Integration:**
```typescript
import { Resend } from 'resend';

const resend = new Resend(process.env.RESEND_API_KEY);

await resend.emails.send({
  from: 'ShopFlow <orders@shopflow.com>',
  to: customerEmail,
  subject: `Order Confirmation #${orderNumber}`,
  react: OrderConfirmationEmail({ order }),
});
```

**Order Confirmation Email Template:**
```tsx
export function OrderConfirmationEmail({ order }: { order: Order }) {
  return (
    <Html>
      <Head />
      <Body style={{ fontFamily: 'Arial, sans-serif' }}>
        <Container>
          <Heading>Thank you for your order!</Heading>
          <Text>Order #{order.number} confirmed</Text>
          <Section>
            <Row>
              <Column>{/* Order items table */}</Column>
            </Row>
          </Section>
          <Text>Estimated delivery: {order.estimatedDelivery}</Text>
          <Button href={trackingUrl}>Track Your Order</Button>
        </Container>
      </Body>
    </Html>
  );
}
```

**Tracking Link Generation:**
```typescript
const carriers = {
  ups: (trackingNumber: string) =>
    `https://www.ups.com/track?tracknum=${trackingNumber}`,
  fedex: (trackingNumber: string) =>
    `https://www.fedex.com/fedextrack/?trknbr=${trackingNumber}`,
  usps: (trackingNumber: string) =>
    `https://tools.usps.com/go/TrackConfirmAction?tLabels=${trackingNumber}`,
};

function getTrackingUrl(carrier: string, trackingNumber: string) {
  return carriers[carrier]?.(trackingNumber) || '#';
}
```

**Print Invoice Functionality:**
```css
@media print {
  header, footer, .no-print {
    display: none;
  }

  .invoice {
    width: 100%;
    max-width: none;
  }
}
```

**Email Testing:**
- Use Resend's email preview feature for design validation
- Test rendering in Gmail, Outlook, Apple Mail, Yahoo Mail
- Validate HTML with Email on Acid or Litmus
- Test deliverability to avoid spam folders (SPF, DKIM, DMARC records)

**Edge Cases:**
- Handle email delivery failures (retry logic, fallback notification)
- Handle missing order data (graceful fallback in template)
- Handle invalid tracking numbers (show placeholder, manual check)
- Handle international addresses (different date/currency formats)
- Handle orders with no shipping (digital products, in-store pickup)
- Handle email bounces (update user email status, notify admin)
- Handle unsubscribe requests (update preferences, exclude from marketing)

## Tasks

- [ ] Create order confirmation page at `/order-confirmation`
- [ ] Fetch order data by orderId from database
- [ ] Build order summary component with all order details
- [ ] Display itemized product list with images and prices
- [ ] Show shipping and billing addresses
- [ ] Add payment method display (masked card number)
- [ ] Implement print invoice functionality with CSS
- [ ] Install and configure Resend SDK
- [ ] Set up Resend API key and domain verification
- [ ] Create OrderConfirmationEmail React component
- [ ] Design email template with responsive HTML/CSS
- [ ] Implement order confirmation email sending on order creation
- [ ] Create ShippingNotificationEmail React component
- [ ] Build shipping notification trigger on order status update
- [ ] Generate carrier tracking links based on carrier type
- [ ] Add email preference center page
- [ ] Test email rendering in major email clients
- [ ] Configure SPF, DKIM, DMARC records for email domain
- [ ] Add retry logic for failed email delivery
- [ ] Create admin view for email delivery logs

## Dependencies

- Story 3-3: Payment Integration (order created after successful payment)
- Story 5-2: Order Management (shipping notification triggers from admin)

## Estimation

**Story Points:** 8

**Rationale:** Email integration with Resend is straightforward, but designing HTML email templates that work across all clients is time-consuming. Order confirmation page is moderate complexity. Tracking link generation and print functionality add scope. Email deliverability setup (DNS records) requires coordination. Overall moderate-to-high complexity with clear implementation path.
