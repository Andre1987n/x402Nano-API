# x402 Nano API Documentation

## Overview

The x402 Nano API is a secure, high-performance **payment gateway** for Nano cryptocurrency. Similar to Stripe or PayPal, we provide a REST API for managing wallets, processing payments, and verifying transactions.

> **About the Name:** "x402" is a brand name inspired by payment-required concepts. This is **not** an implementation of the HTTP 402 Payment Required protocol. Instead, we offer a flexible payment processing API designed for merchants, developers, and AI agents.

**Base URL:** `https://api.x402nano.com`

**Protocol:** HTTPS only  
**Format:** JSON  
**Authentication:** API Key (where applicable)

**What We Offer:**
- 💳 Payment gateway functionality (create/verify transactions)
- 👛 Wallet management (create, import, encrypt)
- 💰 Transaction processing (send, receive, balance checks)
- 🤖 AI-friendly API design (perfect for ChatGPT, Claude integration)
- ⚡ Fast, secure, and developer-friendly

---

## ⚠️ Test Phase Notice

**Current Status: BETA / TEST PHASE**

This API is currently in active development and testing. Please be aware:

- ✅ **Currently:** Most endpoints are publicly accessible for testing
- 🔄 **Future Production:** All endpoints will require API key authentication
- 📚 **Documentation:** Check [GitHub](https://github.com/Andre1987n/x402Nano-API) for latest updates
- 🤖 **AI Integration:** This documentation is designed to be AI-readable for ChatGPT, Claude, and other AI assistants

**For AI Assistants:** Always check this GitHub documentation for the most current API specifications and authentication requirements.

---

## Table of Contents

1. [Authentication](#authentication)
2. [API Endpoints](#api-endpoints)
   - [Create AI API Key](#1-create-ai-api-key)
   - [Create Wallet](#2-create-wallet)
   - [Import Wallet](#3-import-wallet)
   - [Unlock Wallet](#4-unlock-wallet)
   - [Get Balance](#5-get-balance)
   - [Receive Transactions](#6-receive-transactions)
   - [Send Transaction](#7-send-transaction)
   - [Donate](#8-donate)
   - [Create Payment Transaction](#9-create-payment-transaction)
   - [Pay Transaction](#10-pay-transaction)
   - [Get Transaction Status](#11-get-transaction-status)
3. [Webhooks](#webhooks)
4. [Metadata](#metadata)
5. [Error Handling](#error-handling)
6. [Rate Limiting](#rate-limiting)
7. [Security Best Practices](#security-best-practices)

---

## Authentication

**Current (Test Phase):**
- API key required for wallet creation operations
- Other endpoints are publicly accessible for testing
- IP-based rate limiting in effect

**Future (Production):**
- All endpoints will require API key authentication
- Stricter rate limits per API key
- Enhanced security measures

**Required Headers:**
```
Content-Type: application/json
```

**Note:** Your IP address is automatically detected by the server for rate limiting and security purposes. No need to provide it manually.

---

## API Endpoints

### 1. Create AI API Key

Generate a new API key for accessing the x402 Nano API.

**Endpoint:** `POST /key/create`

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{}
```

**Response:**
```
CZtYHgn0Nk8KPvtlkwARjF+m601hC00pqYQwzXaKixU=
```

**Response Format:** Plain text string (the API key itself)

**Note:** Save this API key securely - you'll need it for wallet creation.

**cURL Example:**
```bash
curl -X POST https://api.x402nano.com/key/create \
  -H "Content-Type: application/json" \
  -d '{}'
```

---

### 2. Create Wallet

Create a new Nano wallet with password encryption.

**Endpoint:** `POST /wallet/create`

**⚠️ REQUIRES API KEY:** This endpoint requires a valid API key in the request body.

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{
  "password": "your_secure_password",
  "password_confirmation": "your_secure_password",
  "email": "your_email@example.com",
  "api_key": "your_api_key"
}
```

**Required Fields:**
- `api_key` **(REQUIRED)** - Must be exactly 44 characters. Get one from [Create AI API Key](#1-create-ai-api-key)

**Password Requirements:**
- Must contain at least one uppercase letter
- Must contain at least one special character
- Must match password_confirmation exactly

**Email (Optional but Recommended):**
- If provided, your encrypted wallet backup will be sent to this email
- Prevents data loss if you lose your wallet string
- Must be a valid email format (contains @ and .)

**Response:**
```json
{
  "encrypted_wallet_string": "encrypted_wallet_data",
  "address": "nano_1abc...",
  "public_key": "public_key_hex"
}
```

**cURL Example:**
```bash
curl -X POST https://api.x402nano.com/wallet/create \
  -H "Content-Type: application/json" \
  -d '{
    "password": "MySecurePassword123!",
    "password_confirmation": "MySecurePassword123!",
    "email": "your_email@example.com",
    "api_key": "your_api_key_here"
  }'
```

**Notes:**
- Store the `encrypted_wallet_string` securely - this is required for all future operations
- Password must match password_confirmation
- The wallet is encrypted client-side before storage
- **Recommended:** Provide an email address to receive a backup of your encrypted wallet string
- Email backup prevents data loss if you lose your wallet string (especially useful for AI chat sessions)

---

### 3. Import Wallet

Import an existing Nano wallet using a private seed.

**Endpoint:** `POST /wallet/import`

**⚠️ REQUIRES API KEY:** This endpoint requires a valid API key in the request body.

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{
  "private_wallet_seed": "64_character_hex_seed",
  "password": "your_secure_password",
  "password_confirmation": "your_secure_password",
  "api_key": "your_api_key"
}
```

**Required Fields:**
- `api_key` **(REQUIRED)** - Must be exactly 44 characters. Get one from [Create AI API Key](#1-create-ai-api-key)
- `private_wallet_seed` - Must be exactly 64 characters (hex)

**Response:**
```json
{
  "encrypted_wallet_string": "encrypted_wallet_data",
  "address": "nano_1abc...",
  "public_key": "public_key_hex"
}
```

**cURL Example:**
```bash
curl -X POST https://api.x402nano.com/wallet/import \
  -H "Content-Type: application/json" \
  -d '{
    "private_wallet_seed": "your_64_char_seed_here",
    "password": "MySecurePassword123!",
    "password_confirmation": "MySecurePassword123!",
    "api_key": "your_api_key_here"
  }'
```

---

### 4. Unlock Wallet

Decrypt and unlock a wallet for transaction operations.

**Endpoint:** `POST /wallet/unlock`

**⚠️ API KEY EMBEDDED:** The encrypted wallet string contains an embedded API key that is validated during decryption. You must have created or imported the wallet with a valid API key.

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{
  "encrypted_wallet_string": "your_encrypted_wallet_data",
  "password": "your_password"
}
```

**Response:**
```json
{
  "active_index": 0,
  "address": "nano_33q3tqp8m7toagkj91jtfia4sk8sn1wpars9tk4beiocp3xfmskeztqzygbi",
  "public_key": "86E1D5EC69975543A513823A6C102CC8D9A039646327D4849642AAB07AD9E64C",
  "private_key": "F70D98B3E23FEA69BD7C41D2BBFA131038A85553CB423C0D4B820DC8F2618CFE",
  "wallet_private_seed": "0D09063E701AD0792CFD92D2A218BB3A70FE7504FD7392E42DFBA80859BC3BE5",
  "ai_api_key": "CZtYHgn0Nk8KPvtlkwARjF+m601hC00pqYQwzXaKixU="
}
```

**Warning:** This response contains sensitive information including your private key. Never share this data.

**cURL Example:**
```bash
curl -X POST https://api.x402nano.com/wallet/unlock \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "encrypted_data_here",
    "password": "MySecurePassword123!"
  }'
```

**Notes:**
- Use this endpoint after creating or importing a wallet to get your address
- The response contains your private keys - keep this information secure
- The `ai_api_key` field shows the API key used to create this wallet

---

### 5. Get Balance

Retrieve the balance of a Nano address.

**Endpoint:** `POST /balance/{address}`

**URL Parameters:**
- `address` (string, required): The Nano address to query

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{}
```

**Response:**
```json
{
  "account": "nano_1ebhjnii43rx9fs41njqam86q9uwgcfbap18beoyrpheekw7wniu9e3g6rb3",
  "balance": "7.690001",
  "balance_raw": "7690000600000000000000000000000",
  "pending": "0.000000",
  "pending_raw": "0"
}
```

**cURL Example:**
```bash
curl -X POST https://api.x402nano.com/balance/nano_3i9iuh88bzkgjnt6usuda5763axrnwpm6hce95gai6kn5to8fb3f3fgmia17 \
  -H "Content-Type: application/json" \
  -d '{}'
```

**Notes:**
- `balance` is the human-readable format in NANO
- `balance_raw` is in RAW (smallest Nano unit: 1 NANO = 10^30 RAW)
- `pending` shows pending transactions awaiting receipt
- `pending_raw` is the pending amount in RAW format

---

### 6. Receive Transactions

Process and receive pending transactions for a wallet.

**Endpoint:** `POST /receive`

**⚠️ API KEY EMBEDDED:** The encrypted wallet string contains an embedded API key that is validated during decryption. You must have created or imported the wallet with a valid API key.

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{
  "encrypted_wallet_string": "your_encrypted_wallet_data",
  "password": "your_password"
}
```

**Response:**
```json
{
  "success": true,
  "new_balance": "0.100000",
  "received_from": "nano_1ebhjnii43rx9fs41njqam86q9uwgcfbap18beoyrpheekw7wniu9e3g6rb3"
}
```

**cURL Example:**
```bash
curl -X POST https://api.x402nano.com/receive \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "encrypted_data_here",
    "password": "MySecurePassword123!"
  }'
```

**Notes:**
- This endpoint automatically receives all pending transactions
- Returns the new balance after receiving
- Shows the address that sent you the Nano
- Multiple pending transactions are processed and combined into the new balance

---

### 7. Send Transaction

Send Nano to another address.

**Endpoint:** `POST /send`

**⚠️ API KEY EMBEDDED:** The encrypted wallet string contains an embedded API key that is validated during decryption. You must have created or imported the wallet with a valid API key.

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{
  "encrypted_wallet_string": "your_encrypted_wallet_data",
  "password": "your_password",
  "to_address": "nano_destination_address",
  "amount": "1.5"
}
```

**Response:**
```json
{
  "success": true,
  "send_to": "nano_1ebhjnii43rx9fs41njqam86q9uwgcfbap18beoyrpheekw7wniu9e3g6rb3",
  "ammount": "0.1"
}
```

**cURL Example:**
```bash
curl -X POST https://api.x402nano.com/send \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "encrypted_data_here",
    "password": "MySecurePassword123!",
    "to_address": "nano_3abc123...",
    "amount": "1.5"
  }'
```

**Notes:**
- Amount should be specified in NANO (e.g., "0.1" or "1.5")
- Transactions are processed immediately and irreversible
- Returns success status and destination address
- Verify you have sufficient balance before sending

---

### 8. Donate

Donate Nano to support the x402 Nano API development. This endpoint simplifies the donation process by automatically sending to the official donation address.

**Endpoint:** `POST /donate`

⚠️ **API Key Required:** Must provide valid API key in wallet creation (embedded in `encrypted_wallet_string`)

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{
  "encrypted_wallet_string": "your_encrypted_wallet_string_here",
  "password": "your_secure_password",
  "amount": "0.1"
}
```

**Parameters:**
- `encrypted_wallet_string` (string, required): Your encrypted wallet data from wallet creation/unlock
- `password` (string, required): Your wallet password to decrypt and sign
- `amount` (string, required): Amount to donate in Nano (e.g., "0.1" for 0.1 NANO)

**Response:**
```json
{
  "success": true,
  "message": "Thank you for your donation!"
}
```

**cURL Example:**
```bash
curl -X POST https://api.x402nano.com/donate \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "your_encrypted_wallet_here",
    "password": "YourSecureP@ss123",
    "amount": "0.5"
  }'
```

**PowerShell Example:**
```powershell
Invoke-RestMethod -Uri "https://api.x402nano.com/donate" `
  -Method Post `
  -Headers @{ "Content-Type" = "application/json" } `
  -Body '{
    "encrypted_wallet_string": "your_encrypted_wallet_here",
    "password": "YourSecureP@ss123",
    "amount": "0.5"
  }'
```

**Notes:**
- Donation address: `nano_3ep6fd8p1zcypgi3fsm7x3jr7n8qj57tk5ahpseqynaq7uafzo68bk6tci9n`
- Works exactly like `/send` but automatically uses the donation address
- Supports any amount in Nano
- Transaction is processed immediately
- Donations help maintain and improve the API infrastructure
- Same security requirements as send endpoint (password validation, API key)

**Error Response (Invalid Password):**
```json
{
  "error": "invalid_password",
  "message": "Invalid password provided"
}
```

**Error Response (Insufficient Balance):**
```json
{
  "error": "insufficient_balance",
  "message": "Insufficient balance for this transaction"
}
```

---

### 9. Create Payment Transaction

Create a temporary payment transaction for users to pay. Returns a unique transaction ID and payment details that expire after 60 minutes.

**NEW in v0.2.0:** Optional `callback_url` for webhook notifications and `metadata` for custom data.

**Endpoint:** `POST /transaction/create`

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{
  "receive_address": "nano_1your_server_address",
  "amount": "1.5",
  "callback_url": "https://yoursite.com/webhook",
  "metadata": "{\"order_id\":\"ORD-123\",\"customer\":\"user@example.com\"}"
}
```

**Fields:**
- `receive_address` (required) - Your Nano address to receive payment
- `amount` (required) - Amount in Nano (e.g., "1.5")
- `callback_url` (optional) - Your webhook URL for instant payment notifications
- `metadata` (optional) - JSON string with custom data (max 4KB, must be valid JSON)

**Response:**
```json
{
  "receive_address": "nano_1your_server_address",
  "amount": "1.500000234",
  "transaction_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

> **⚠️ Important - Unique Amount:** Notice the amount changed from "1.5" to "1.500000234". The system automatically adds a microscopic offset (derived from the transaction UUID) to ensure uniqueness. This prevents payment conflicts when multiple users create transactions with the same amount simultaneously.
>
> **How it works:**
> - Original amount: `1.5` NANO
> - Unique offset: `0.000000234` (max 0.000001 NANO)
> - Final amount: `1.500000234` NANO
>
> **Why?** If two AI agents request payment for "1 NANO" at the same time, the blockchain can distinguish between them (1.000000123 vs 1.000000456). This enables accurate payment matching without conflicts.
>
> **Use the returned amount:** Always use the `amount` value from the response when paying or checking status, not your original requested amount.

**cURL Example:**
```bash
curl -X POST https://api.x402nano.com/transaction/create \
  -H "Content-Type: application/json" \
  -d '{
    "receive_address": "nano_1your_server_address",
    "amount": "1.5"
  }'
```

**Notes:**
- Transaction expires after 60 minutes
- Expired transactions are automatically cleaned up every 5 minutes
- Transaction ID is a UUID v4
- Save the transaction_id to check payment status later

---

### 10. Pay Transaction

Process payment for a previously created transaction using encrypted wallet credentials.

**Endpoint:** `POST /transaction/pay`

**⚠️ API KEY EMBEDDED:** The encrypted wallet string contains an embedded API key that is validated during decryption. You must have created or imported the wallet with a valid API key.

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Request Body:**
```json
{
  "transaction_id": "550e8400-e29b-41d4-a716-446655440000",
  "encrypted_wallet_string": "your_encrypted_wallet_data",
  "password": "your_password",
  "amount": "1.500000234"
}
```

**⚠️ Critical:** The `amount` must match the **exact unique amount** returned during transaction creation (e.g., "1.500000234"), not your original requested amount ("1.5"). This ensures payment goes to the correct transaction.

**Response:**
```json
{
  "success": "true",
  "transaction_id": "550e8400-e29b-41d4-a716-446655440000",
  "to_address": "nano_1your_server_address",
  "amount": "1.500000234"
}
```

**cURL Example:**
```bash
curl -X POST https://api.x402nano.com/transaction/pay \
  -H "Content-Type: application/json" \
  -d '{
    "transaction_id": "550e8400-e29b-41d4-a716-446655440000",
    "encrypted_wallet_string": "encrypted_data_here",
    "password": "MySecurePassword123!",
    "amount": "1.5"
  }'
```

**Error Responses:**
```json
{
  "error": "invalid_transaction_id",
  "message": "Invalid transaction ID"
}
```

```json
{
  "error": "transaction_expired",
  "message": "Transaction has expired. Please create a new transaction."
}
```

```json
{
  "error": "amount_mismatch",
  "message": "Amount does not match the created transaction amount. Amount should be 1.500000234."
}
```

**Notes:**
- Amount must **exactly** match the unique amount returned from `/transaction/create` (e.g., "1.500000234")
- Do not use your original requested amount ("1.5") - use the response value
- Transaction must not be expired (< 60 minutes old)
- Once paid, transaction is moved from pending to paid cache
- Payment is irreversible

---

### 11. Get Transaction Status

Check the payment status of a transaction with long-polling support (waits up to 30 seconds for payment confirmation).

**Endpoint:** `GET /transaction/status/{transaction_id}`

**Request Headers:**
```json
{
  "Content-Type": "application/json"
}
```

**Response (Paid):**
```json
{
  "success": "true",
  "message": "Transaction has been paid.",
  "transaction_id": "550e8400-e29b-41d4-a716-446655440000",
  "is_paid": true,
  "metadata": "{\"order_id\":\"ORD-123\",\"customer\":\"user@example.com\"}"
}
```

**Note:** `metadata` field is included if it was provided during transaction creation.

**Response (Not Found):**
```json
{
  "success": "false",
  "message": "Transaction ID not found.",
  "transaction_id": "550e8400-e29b-41d4-a716-446655440000",
  "is_paid": false
}
```

**Response (Timeout):**
```json
{
  "success": "false",
  "message": "Transaction has not been paid within the timeout period. Try again later.",
  "transaction_id": "550e8400-e29b-41d4-a716-446655440000",
  "is_paid": false
}
```

**cURL Example:**
```bash
curl -X GET https://api.x402nano.com/transaction/status/550e8400-e29b-41d4-a716-446655440000 \
  -H "Content-Type: application/json"
```

**Notes:**
- Long-polling: Request waits up to 30 seconds for payment
- Checks payment status every 500ms
- Returns immediately if transaction is paid or not found
- Returns after 30 seconds if still pending
- Ideal for payment confirmation pages

---

## Error Handling

All endpoints return standard HTTP status codes:

### Success Codes
- `200 OK` - Request successful
- `201 Created` - Resource created successfully

### Error Codes
- `400 Bad Request` - Invalid request parameters
- `401 Unauthorized` - Missing or invalid API key
- `403 Forbidden` - Access denied
- `404 Not Found` - Endpoint or resource not found
- `429 Too Many Requests` - Rate limit exceeded
- `500 Internal Server Error` - Server error

### Error Response Format
```json
{
  "error": "weak_password",
  "message": "Password must contain at least one uppercase letter"
}
```

### Deep Nested Error Handling

The API uses a **deep nested error handling** approach where errors are constructed at the point of failure:

- Errors originate from the lowest level (e.g., RPC calls, database operations)
- Each layer adds context as the error propagates up
- This provides detailed, actionable error messages that help identify exactly where and why something failed

**Example Error Chain:**
```
RPC Call Failed → Parse Error → API Error Response
"Failed to parse RPC response: Failed to parse accounts_receivable response: unexpected field..."
```

This approach ensures:
- **Precise error location**: Know exactly which operation failed
- **Full context**: See the complete error chain from source to API response
- **Better debugging**: Detailed messages help developers troubleshoot issues quickly

**Common Error Messages:**
- `"weak_password"` - Password doesn't meet requirements
- `"invalid_credentials"` - Wrong password or encrypted wallet string
- `"insufficient_balance"` - Not enough balance to send
- `"invalid_address"` - Malformed Nano address
- `"rpc_error"` - Nano node RPC call failed (includes detailed context)
- `"rpc_response_parse_failed"` - Failed to parse RPC response (includes parse details)
- `"database_error"` - Database operation failed
- `"invalid_api_key"` - API key is invalid or missing (required for wallet operations)

---

## Rate Limiting

The API implements rate limiting to ensure fair usage:

- **Default Limit:** 100 requests per minute per IP address
- **API Key Limit:** Higher limits available with API key authentication

When rate limited, the API returns:
```json
{
  "error": "Rate limit exceeded",
  "retry_after": 60
}
```

**Headers:**
- `X-RateLimit-Limit`: Maximum requests allowed
- `X-RateLimit-Remaining`: Requests remaining in current window
- `X-RateLimit-Reset`: Unix timestamp when limit resets

---

## Security Best Practices

### For Developers

1. **Never expose your API key** in client-side code or public repositories
2. **Use HTTPS only** - the API enforces TLS/SSL
3. **Store encrypted_wallet_string securely** - treat it like a private key
4. **Use strong passwords** for wallet encryption
5. **Implement request signing** for sensitive operations
6. **Rotate API keys regularly**

### Wallet Security

- **Encrypted Storage:** Wallets are encrypted with AES-256
- **Password Protection:** All wallet operations require password verification
- **No Private Key Exposure:** Private keys never leave the encrypted container
- **Secure Transmission:** All data transmitted over HTTPS

### Best Practices for AI Integration

When integrating this API with AI systems:

1. **Environment Variables:** Store API keys and sensitive data in environment variables
2. **Request Validation:** Always validate user input before making API calls
3. **Error Handling:** Implement comprehensive error handling for all API responses
4. **Transaction Confirmation:** Always confirm transaction details with users before sending
5. **Balance Checks:** Verify sufficient balance before attempting sends
6. **Audit Logging:** Log all API interactions for security auditing

---

## Code Examples

### Python Example
```python
import requests
import json

BASE_URL = "https://api.x402nano.com"

def create_api_key():
    headers = {
        "Content-Type": "application/json"
    }
    
    response = requests.post(
        f"{BASE_URL}/key/create",
        headers=headers,
        json={}
    )
    
    return response.text  # Returns plain text API key

def create_wallet(api_key, password):
    headers = {
        "Content-Type": "application/json"
    }
    
    data = {
        "password": password,
        "password_confirmation": password,
        "api_key": api_key
    }
    
    response = requests.post(
        f"{BASE_URL}/wallet/create",
        headers=headers,
        json=data
    )
    
    return response.text  # Returns plain text encrypted wallet

def unlock_wallet(encrypted_wallet, password):
    headers = {
        "Content-Type": "application/json"
    }
    
    data = {
        "encrypted_wallet_string": encrypted_wallet,
        "password": password
    }
    
    response = requests.post(
        f"{BASE_URL}/wallet/unlock",
        headers=headers,
        json=data
    )
    
    return response.json()

def send_nano(encrypted_wallet, password, to_address, amount):
    headers = {
        "Content-Type": "application/json"
    }
    
    data = {
        "encrypted_wallet_string": encrypted_wallet,
        "password": password,
        "to_address": to_address,
        "amount": amount
    }
    
    response = requests.post(
        f"{BASE_URL}/send",
        headers=headers,
        json=data
    )
    
    return response.json()

def receive_nano(encrypted_wallet, password):
    headers = {
        "Content-Type": "application/json"
    }
    
    data = {
        "encrypted_wallet_string": encrypted_wallet,
        "password": password
    }
    
    response = requests.post(
        f"{BASE_URL}/receive",
        headers=headers,
        json=data
    )
    
    return response.json()
```

### JavaScript/Node.js Example
```javascript
const axios = require('axios');

const BASE_URL = 'https://api.x402nano.com';

async function createApiKey() {
    const headers = {
        'Content-Type': 'application/json'
    };
    
    const response = await axios.post(
        `${BASE_URL}/key/create`,
        {},
        { headers }
    );
    
    return response.data;  // Returns plain text API key
}

async function createWallet(apiKey, password) {
    const headers = {
        'Content-Type': 'application/json'
    };
    
    const data = {
        password: password,
        password_confirmation: password,
        api_key: apiKey
    };
    
    const response = await axios.post(
        `${BASE_URL}/wallet/create`,
        data,
        { headers }
    );
    
    return response.data;  // Returns plain text encrypted wallet
}

async function unlockWallet(encryptedWallet, password) {
    const headers = {
        'Content-Type': 'application/json'
    };
    
    const data = {
        encrypted_wallet_string: encryptedWallet,
        password: password
    };
    
    const response = await axios.post(
        `${BASE_URL}/wallet/unlock`,
        data,
        { headers }
    );
    
    return response.data;
}

async function getBalance(address) {
    const headers = {
        'Content-Type': 'application/json'
    };
    
    const response = await axios.post(
        `${BASE_URL}/balance/${address}`,
        {},
        { headers }
    );
    
    return response.data;
}

async function receiveNano(encryptedWallet, password) {
    const headers = {
        'Content-Type': 'application/json'
    };
    
    const data = {
        encrypted_wallet_string: encryptedWallet,
        password: password
    };
    
    const response = await axios.post(
        `${BASE_URL}/receive`,
        data,
        { headers }
    );
    
    return response.data;
}

async function sendNano(encryptedWallet, password, toAddress, amount) {
    const headers = {
        'Content-Type': 'application/json'
    };
    
    const data = {
        encrypted_wallet_string: encryptedWallet,
        password: password,
        to_address: toAddress,
        amount: amount
    };
    
    const response = await axios.post(
        `${BASE_URL}/send`,
        data,
        { headers }
    );
    
    return response.data;
}
```

---

## Webhooks

### Overview

Webhooks provide instant push notifications when transactions are paid. Instead of polling the status endpoint every few seconds, your server receives an HTTP POST request immediately when payment is confirmed.

**Benefits:**
- ⚡ Instant notifications (< 1 second after payment)
- 📉 Reduced server load (no polling needed)
- 🤖 Perfect for AI agents and automation
- 🔄 Automatic retry on delivery failure

### Setup

1. **Create transaction with callback_url:**
```json
POST /transaction/create
{
  "receive_address": "nano_...",
  "amount": "1.5",
  "callback_url": "https://yoursite.com/webhook"
}
```

2. **Implement webhook endpoint:**
```python
from flask import Flask, request
import hmac
import hashlib

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    # Get transaction ID from headers
    transaction_id = request.headers.get('X-Transaction-Id', '')
    
    # Get payload
    data = request.json
    
    # Process payment
    order_id = data.get('metadata', {}).get('order_id')
    amount = data['amount']
    
    print(f"Payment received for order {order_id}: {amount} Nano")
    # Mark order as paid, start shipping, etc.
    
    return 'OK', 200
```

3. **Whitelist our server IP (recommended):**
```nginx
# In your firewall/nginx config
location /webhook {
    allow 57.129.16.77;  # x402 Nano server
    deny all;
}
```

### Webhook Payload

**Headers:**
```
Content-Type: application/json
X-Transaction-Id: uuid-123
```

**Body:**
```json
{
  "transaction_id": "550e8400-e29b-41d4-a716-446655440000",
  "receive_address": "nano_1your_server_address",
  "amount": "1.5",
  "metadata": "{\"order_id\":\"ORD-123\"}",
  "paid_at": "2026-01-10T14:30:00Z"
}
```

### Retry Logic

If webhook delivery fails, we automatically retry:
- **Attempt 1:** Immediate
- **Attempt 2:** 30 seconds later
- **Attempt 3:** 5 minutes later

**Success criteria:** HTTP status 200-299

**Logged:** All delivery attempts are logged in database for debugging

### Security

**IP Whitelisting (Recommended)**
- Our server IP: `57.129.16.77`
- Configure your firewall to only accept webhooks from this IP
- This is the most secure and simplest method

**Example verification (Python):**
```python
from flask import Flask, request

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    # Verify IP (optional - can also do at firewall level)
    client_ip = request.headers.get('X-Forwarded-For', request.remote_addr)
    if client_ip != '57.129.16.77':
        return 'Unauthorized', 403
    
    # Process webhook
    data = request.json
    print(f"Payment received: {data['amount']} Nano")
    
    return 'OK', 200
```

---

## Metadata

### Overview

Metadata allows you to attach custom JSON data to transactions. This is perfect for tracking orders, customers, sessions, or any other contextual information.

**Use Cases:**
- 🛒 E-commerce: Order IDs, customer emails, product details
- 🤖 AI Services: Session IDs, service types, user identifiers
- 📊 API Metering: Endpoint names, usage tiers, billing periods
- 🎮 Gaming: Player IDs, item purchases, in-game transactions

### Usage

**Create transaction with metadata:**
```json
POST /transaction/create
{
  "receive_address": "nano_...",
  "amount": "1.5",
  "metadata": "{\"order_id\":\"ORD-123\",\"customer\":\"user@example.com\",\"items\":[\"License\",\"Support\"]}"
}
```

**Retrieve in status check:**
```json
GET /transaction/status/{id}
{
  "success": "true",
  "is_paid": true,
  "metadata": "{\"order_id\":\"ORD-123\",\"customer\":\"user@example.com\"}"
}
```

**Receive in webhook:**
```json
POST https://yoursite.com/webhook
{
  "transaction_id": "...",
  "amount": "1.5",
  "metadata": "{\"order_id\":\"ORD-123\"}"
}
```

### Limitations

- **Max size:** 4KB (4096 bytes)
- **Format:** Must be valid JSON string
- **Validation:** Checked on creation, returns error if invalid
- **Storage:** Persisted in PostgreSQL database

### Examples

**E-commerce Order:**
```json
{
  "metadata": "{\"order_id\":\"ORD-12345\",\"customer_email\":\"buyer@example.com\",\"product\":\"Premium License\",\"quantity\":1,\"discount_code\":\"SAVE10\"}"
}
```

**AI Service:**
```json
{
  "metadata": "{\"session_id\":\"sess_abc123\",\"service\":\"GPT-4\",\"model\":\"gpt-4-turbo\",\"tokens\":1500,\"user_id\":\"user_xyz\"}"
}
```

**API Metering:**
```json
{
  "metadata": "{\"api_key\":\"key_123\",\"endpoint\":\"/v1/data\",\"requests\":1000,\"period\":\"2026-01\"}"
}
```

---

## Support and Resources

- **GitHub Repository:** [x402 Nano API](https://github.com/Andre1987n/x402Nano-API)
- **Nano Documentation:** [docs.nano.org](https://docs.nano.org)
- **Issue Tracking:** [GitHub Issues](https://github.com/Andre1987n/x402Nano-API/issues)
- **Server IP:** 57.129.16.77 (for webhook whitelisting)

---

## Changelog

### Version 0.2.0 (2026-01-10)
- 🎉 **NEW:** Webhook support with automatic retry logic
- 🎉 **NEW:** Metadata field for custom transaction data (max 4KB)
- 🎉 **NEW:** PostgreSQL persistence for paid transactions
- 🎉 **NEW:** Webhook delivery logging
- ✨ HMAC-SHA256 signature for webhook verification
- ✨ Complete audit trail for all payments

### Version 0.1.0 (2026-01-05)
- Initial public release
- Core wallet management endpoints
- Transaction processing (send/receive)
- Balance queries
- API key authentication
- Password strength requirements (uppercase + special character)
- Tested end-to-end transaction flow

**Complete Tested Workflow:**
1. Create API Key → Returns plain text key
2. Create Wallet → Returns encrypted wallet string
3. Unlock Wallet → Returns address, keys, and API key
4. Receive Transactions → Processes pending blocks and updates balance
5. Send Transactions → Transfers Nano with immediate confirmation
6. Check Balance → Verifies transaction completion

---

## License

This API is provided as-is for use with the x402 Nano platform. Please review the full license terms in the repository.
