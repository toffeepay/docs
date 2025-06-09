---
sidebar_position: 1
---

# Getting Started

This guide walks you through:
1. [Authentication](#authentication)
2. [Creating a payment session](#create-a-payment-session)
3. [Opening the payment page](#open-the-payment-page)
4. [Handling payment confirmation via webhook](#handle-payment-webhook)
5. [Resuming the game`](#return-to-game)
6. [Confirming payment](#confirming-payment)

For a visual overview of the complete payment flow, see the [Payments](/payments#payment-flow) page.

---

## Authentication

All requests to the ToffeePay API must include your API key. See the [Authentication](/authentication) page for details.

---

## Create a Payment Session

Send a server-side request to create a payment session. See the [Payments](/payments#create-a-payment-session) page for detailed API documentation.

**Quick Example:**
```http
POST /pay.v1.PaymentService/CreateSession
{
  "game_id": "space_shooter",
  "user_id": "player_42",
  "item": { "title": "50 Gold Coins", "price": 499, "currency": "USD" },
  "return_url": "mygame://payment-complete"
}
```

---

## Open the Payment Page

Open the returned payment `url` in the browser.

The user will:
- View the list of items and total price
- Pay using Apple Pay or another supported method

---

## Handle Payment Webhook

Once payment is successful, ToffeePay sends a **signed webhook** to your backend. This is the most reliable way to process payments.

See the [Webhooks](/webhooks) page for complete implementation details including signature verification.

---

## Return to Game

After a successful payment, the player is **redirected to your specified `return_url`**.

This is typically a **custom deep link** that your game handles to:
- Show a success screen
- Confirm the session was paid

**Example:**
```json
"return_url": "mygame://payment-complete"
```

Make sure your mobile client is set up to **handle and parse this URI scheme**.

---

### Confirming Payment

To confirm the payment status after returning to the game, call `GetSessionStatus` using the existing `session_id`.

See the [Payments](/payments#check-session-status) page for detailed API documentation and status handling.

