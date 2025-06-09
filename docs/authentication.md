---
sidebar_position: 2
---

# Authentication

All requests to the ToffeePay API must include your API key:

```http
Authorization: Bearer <your_api_key>
```

You'll receive this key when you register your game. Keep it secret and **only use it server-side**.

## Best Practices

1. **Server-side only**: Never expose API keys in client-side code
2. **Security**: Use HTTPS for all API requests
3. **Testing**: Use sandbox API keys (different from production) for development

## Error Handling

Common authentication errors:

- `invalid_game_id`: Check your game registration and API key
- `unauthorized`: Verify your API key is correct and hasn't expired