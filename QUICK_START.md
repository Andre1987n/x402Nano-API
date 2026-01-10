# Quick Start Guide - x402 Nano API

This guide will help you get started with the x402 Nano API in minutes.

---

## ⚠️ Important: Test Phase

**This API is currently in BETA/TEST PHASE**

- Most endpoints are currently open for testing without strict API key requirements
- **In production:** All endpoints will require API key authentication
- API behavior may change - check [GitHub](https://github.com/Andre1987n/x402Nano-API) for updates
- Designed for AI assistant integration (ChatGPT, Claude, etc.)

---

## Prerequisites

- HTTPS client (curl, Postman, or programming language HTTP library)
- Basic understanding of REST APIs and JSON

**Note:** Your IP address is automatically detected by the server - no need to send it manually!

## Step 1: Get an API Key

First, create an API key:

```bash
curl -X POST https://api.x402nano.com/key/create \
  -H "Content-Type: application/json" \
  -d '{}'
```

**Response:**
```
CZtYHgn0Nk8KPvtlkwARjF+m601hC00pqYQwzXaKixU=
```

**Save this API key** - you'll need it for wallet operations.

**Note:** The response is a plain text API key string, not JSON.

## Step 2: Create a Wallet

Create a new Nano wallet:

**⚠️ IMPORTANT:** You must include the API key from Step 1. This API key will be embedded in your encrypted wallet and required for all future wallet operations (unlock, send, receive).

```bash
curl -X POST https://api.x402nano.com/wallet/create \
  -H "Content-Type: application/json" \
  -d '{
    "password": "YourSecurePassword123!",
    "password_confirmation": "YourSecurePassword123!",
    "email": "your_email@example.com",
    "api_key": "your_api_key_from_step_1"
  }'
```

**Password Requirements:**
- Must contain at least one uppercase letter
- Must contain at least one lowercase letter
- Must contain at least one digit
- Must contain at least one special character
- Must match password_confirmation

**API Key Requirements:****
- Must be exactly 44 characters
- Required for wallet creation, import, and all transaction operations
- Gets embedded in your encrypted wallet data

**Email (Optional but Recommended):**
- Providing an email address will send you a backup of your encrypted wallet
- Prevents data loss if you lose your wallet string
- Especially useful for AI chat sessions that may close unexpectedly

**Response:**
```
NihweB70VlrFi5nSFzJkQonXzSJhJZVaqogIoODKTdtwoFDLJy2AsAjaSgycsOhd59txyP1XR+j/4ffJBnUjgtZZg4Y73Afv6q7kY4YHoUd1nJxVqDStK602rRqhik47epY0dgrYAGFAGSEz10Da7aEpfr7wdLzoYkQVdohPcgizgjzMARtnaea0rZ625X96h9dNnI9MzCnIsSdXKWu5fzLZ...
```

**Important:** Save the encrypted wallet string. If you provided an email, you'll also receive a backup via email. To get your wallet address, unlock it:

```bash
curl -X POST https://api.x402nano.com/wallet/unlock \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "YOUR_ENCRYPTED_WALLET",
    "password": "YourSecurePassword123!"
  }'
```

**Unlock Response:**
```json
{
  "active_index": 0,
  "address": "nano_33q3tqp8m7toagkj91jtfia4sk8sn1wpars9tk4beiocp3xfmskeztqzygbi",
  "public_key": "86E1D5EC...",
  "private_key": "F70D98B3...",
  "wallet_private_seed": "0D09063E...",
  "ai_api_key": "CZtYHgn0..."
}
```

## Step 3: Check Balance

Check your wallet balance:

```bash
curl -X POST https://api.x402nano.com/balance/nano_YOUR_ADDRESS \
  -H "Content-Type: application/json" \
  -d '{}'
```

**Response:**
```json
{
  "account": "nano_33q3tqp8m7toagkj91jtfia4sk8sn1wpars9tk4beiocp3xfmskeztqzygbi",
  "balance": "0.000000",
  "balance_raw": "0",
  "pending": "0.000000",
  "pending_raw": "0"
}
```

## Step 4: Receive Pending Transactions

If you have pending transactions:

```bash
curl -X POST https://api.x402nano.com/receive \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "YOUR_ENCRYPTED_WALLET",
    "password": "YourSecurePassword123!"
  }'
```

**Response:**
```json
{
  "success": true,
  "new_balance": "0.100000",
  "received_from": "nano_1ebhjnii43rx9fs41njqam86q9uwgcfbap18beoyrpheekw7wniu9e3g6rb3"
}
```

## Step 5: Send Nano

Send Nano to another address:

```bash
curl -X POST https://api.x402nano.com/send \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "YOUR_ENCRYPTED_WALLET",
    "password": "YourSecurePassword123!",
    "to_address": "nano_3destination_address",
    "amount": "1.5"
  }'
```

**Response:**
```json
{
  "success": true,
  "send_to": "nano_3destination_address",
  "ammount": "1.5"
}
```

## Step 6: Donate to Support Development (Optional)

Support the x402 Nano API development with a quick donation:

```bash
curl -X POST https://api.x402nano.com/donate \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "YOUR_ENCRYPTED_WALLET",
    "password": "YourSecurePassword123!",
    "amount": "0.5"
  }'
```

**Response:**
```json
{
  "success": true,
  "message": "Thank you for your donation!"
}
```

**Notes:**
- Automatically sends to the official donation address
- Works exactly like `/send` but simpler
- Any amount is appreciated
- Helps maintain and improve the API

---

## Common Workflows

### Workflow 1: Create and Fund a New Wallet

1. Create API key
2. Create wallet
3. Share wallet address to receive funds
4. Receive pending transactions
5. Check balance

### Workflow 2: Import Existing Wallet

1. Create API key
2. Import wallet with seed
3. Receive pending transactions
4. Check balance
5. Send transactions

### Workflow 3: Check Balance Only

1. Get wallet address
2. Call balance endpoint (no API key needed)

## Testing with Postman

### Import Collection

Create a Postman collection with these variables:
- `base_url`: `https://api.x402nano.com`
- `api_key`: Your API key
- `encrypted_wallet`: Your encrypted wallet string
- `password`: Your wallet password

### Example Request

**Create Wallet:**
```
POST {{base_url}}/wallet/create
Headers:
  Content-Type: application/json
Body:
{
  "password": "{{password}}",
  "password_confirmation": "{{password}}",
  "api_key": "{{api_key}}"
}
```

## Python Quick Start

```python
import requests

BASE_URL = "https://api.x402nano.com"

# 1. Create API Key
response = requests.post(
    f"{BASE_URL}/key/create",
    headers={"Content-Type": "application/json"},
    json={}
)
api_key = response.text  # Plain text response
print(f"API Key: {api_key}")

# 2. Create Wallet (password requires uppercase + special char)
response = requests.post(
    f"{BASE_URL}/wallet/create",
    headers={"Content-Type": "application/json"},
    json={
        "password": "SecurePass123!",
        "password_confirmation": "SecurePass123!",
        "api_key": api_key
    }
)
encrypted_wallet = response.text  # Plain text response
print(f"Encrypted Wallet: {encrypted_wallet}")

# 3. Unlock Wallet to get address
response = requests.post(
    f"{BASE_URL}/wallet/unlock",
    headers={"Content-Type": "application/json"},
    json={
        "encrypted_wallet_string": encrypted_wallet,
        "password": "SecurePass123!"
    }
)
wallet = response.json()
print(f"Address: {wallet['address']}")

# 4. Check Balance
response = requests.post(
    f"{BASE_URL}/balance/{wallet['address']}",
    headers={"Content-Type": "application/json"},
    json={}
)
balance = response.json()
print(f"Balance: {balance['balance']} NANO")
```

## JavaScript Quick Start

```javascript
const axios = require('axios');

const BASE_URL = 'https://api.x402nano.com';

async function quickStart() {
    // 1. Create API Key
    const keyResponse = await axios.post(
        `${BASE_URL}/key/create`,
        {},
        { headers: { 'Content-Type': 'application/json' } }
    );
    const apiKey = keyResponse.data;  // Plain text response
    console.log('API Key:', apiKey);

    // 2. Create Wallet (password requires uppercase + special char)
    const walletResponse = await axios.post(
        `${BASE_URL}/wallet/create`,
        {
            password: 'SecurePass123!',
            password_confirmation: 'SecurePass123!',
            api_key: apiKey
        },
        { headers: { 'Content-Type': 'application/json' } }
    );
    const encryptedWallet = walletResponse.data;  // Plain text response
    console.log('Encrypted Wallet:', encryptedWallet);

    // 3. Unlock Wallet to get address
    const unlockResponse = await axios.post(
        `${BASE_URL}/wallet/unlock`,
        {
            encrypted_wallet_string: encryptedWallet,
            password: 'SecurePass123!'
        },
        { headers: { 'Content-Type': 'application/json' } }
    );
    const wallet = unlockResponse.data;
    console.log('Address:', wallet.address);

    // 4. Check Balance
    const balanceResponse = await axios.post(
        `${BASE_URL}/balance/${wallet.address}`,
        {},
        { headers: { 'Content-Type': 'application/json' } }
    );
    const balance = balanceResponse.data;
    console.log('Balance:', balance.balance, 'NANO');
}

quickStart().catch(console.error);
```

## Error Handling

### Deep Nested Errors

The API uses a **deep nested error handling** approach. Errors are constructed at the point of failure and include full context:

```json
{
  "error": "rpc_response_parse_failed",
  "message": "Failed to parse RPC response: Failed to parse accounts_receivable response: unexpected field at line 1 column 42"
}
```

**Benefits:**
- Errors show exactly where and why something failed
- Complete error chain from source to API response
- Makes debugging easier with detailed context

### Common Error Codes

- `weak_password` - Password doesn't meet requirements (uppercase, lowercase, digit, special char)
- `invalid_api_key` - API key is invalid, missing, or not exactly 44 characters
- `invalid_credentials` - Wrong password or encrypted wallet string
- `insufficient_balance` - Not enough balance to send
- `invalid_address` - Malformed Nano address
- `rpc_error` - Nano node RPC call failed (includes detailed context)
- `rpc_response_parse_failed` - Failed to parse RPC response
- `transaction_expired` - Payment transaction has expired (>60 minutes)
- `amount_mismatch` - Payment amount doesn't match created transaction

### API Key Requirements Summary

**⚠️ All wallet operations require a valid API key:**

- **Create Wallet** - API key must be provided in request body (44 characters)
- **Import Wallet** - API key must be provided in request body (44 characters)
- **Unlock Wallet** - API key is embedded in encrypted wallet (validated on decrypt)
- **Send Transaction** - API key is embedded in encrypted wallet (validated on decrypt)
- **Receive Transaction** - API key is embedded in encrypted wallet (validated on decrypt)
- **Pay Transaction** - API key is embedded in encrypted wallet (validated on decrypt)

The API key is embedded in your encrypted wallet data during creation/import and automatically validated whenever you decrypt the wallet.

## Troubleshooting

### Common Issues

**Issue:** `400 Bad Request - Password mismatch`  
**Solution:** Verify `password` and `password_confirmation` are identical

**Issue:** `404 Not Found`  
**Solution:** Check the endpoint URL and HTTP method (should be POST)

**Issue:** `429 Too Many Requests`  
**Solution:** You've hit the rate limit. Wait 60 seconds and try again

### Getting Help

- Check the full [API Documentation](API_DOCUMENTATION.md)
- Review error messages carefully
- Verify all required fields are present
- Ensure JSON formatting is correct

## Security Reminders

✅ **DO:**
- Use HTTPS only
- Store API keys securely
- Use strong passwords
- Keep encrypted_wallet_string private
- Validate all inputs

❌ **DON'T:**
- Share your API key publicly
- Commit API keys to version control
- Use weak passwords
- Expose encrypted_wallet_string
- Skip error handling

## Next Steps

- Read the complete [API Documentation](API_DOCUMENTATION.md)
- Review [Security Best Practices](API_DOCUMENTATION.md#security-best-practices)
- Explore advanced features
- Join the community

---

**Need Help?** Open an issue on [GitHub](https://github.com/Andre1987n/x402Nano-API/issues)
