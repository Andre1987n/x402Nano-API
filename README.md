# x402 Nano API - Public Documentation

> ⚠️ **BETA / TEST PHASE** ⚠️
> 
> This API is currently in **test/beta phase**. Most endpoints are currently publicly accessible for testing purposes.
> 
> **Future Changes:**
> - All endpoints will require API key authentication in production
> - Rate limits will be enforced more strictly
> - Additional security measures will be implemented
> 
> Please check this documentation regularly for updates: [GitHub Repository](https://github.com/Andre1987n/x402Nano-API)

Welcome to the x402 Nano API documentation. This API provides a secure, easy-to-use interface for managing Nano cryptocurrency wallets and transactions.

## 📚 Documentation Index

### Getting Started
- **[Quick Start Guide](QUICK_START.md)** - Get up and running in 5 minutes
- **[API Documentation](API_DOCUMENTATION.md)** - Complete API reference

### Key Features

✅ **Wallet Management**
- Create new wallets with password encryption
- Import existing wallets from seed
- Secure wallet unlock mechanism
- Optional email backup for wallet recovery

✅ **Transaction Processing**
- Send Nano to any address
- Quick donate endpoint for API support
- Receive pending transactions automatically
- Real-time balance queries
- AI agent payment requests (create/pay/status)
- Long-polling payment confirmation (30s timeout)
- Auto-expiring transactions (60 minutes)

✅ **Security First**
- AES-256 wallet encryption
- HTTPS-only communication
- No private key exposure
- Rate limiting and DDoS protection

✅ **Developer Friendly**
- RESTful JSON API
- Deep nested error handling with full context
- Comprehensive error messages
- Code examples in multiple languages
- AI-friendly documentation

✅ **API Key Security**
- All wallet operations require a valid API key (44 characters)
- API keys are embedded in encrypted wallet data
- Automatic validation on all encrypt/decrypt operations

## 🚀 Quick Example

```bash
# 1. Create an API key
curl -X POST https://api.x402nano.com/api/key/create \
  -H "Content-Type: application/json" \
  -d '{}'
# Returns: CZtYHgn0Nk8KPvtlkwARjF+m601hC00pqYQwzXaKixU=

# 2. Create a wallet (password requires uppercase + special char)
curl -X POST https://api.x402nano.com/wallet/create \
  -H "Content-Type: application/json" \
  -d '{
    "password": "SecurePassword123!",
    "password_confirmation": "SecurePassword123!",
    "email": "backup@example.com",
    "api_key": "CZtYHgn0Nk8KPvtlkwARjF+m601hC00pqYQwzXaKixU="
  }'
# Returns: Encrypted wallet string (also sent to email for backup)

# 3. Unlock wallet to get your address
curl -X POST https://api.x402nano.com/wallet/unlock \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "YOUR_ENCRYPTED_WALLET",
    "password": "SecurePassword123!"
  }'
# Returns: {"address": "nano_33q3tqp8m7...", ...}

# 4. Check balance
curl -X POST https://api.x402nano.com/balance/nano_YOUR_ADDRESS \
  -H "Content-Type: application/json" \
  -d '{}'

# 5. Receive pending transactions
curl -X POST https://api.x402nano.com/receive \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "YOUR_ENCRYPTED_WALLET",
    "password": "SecurePassword123!"
  }'

# 6. Send Nano
curl -X POST https://api.x402nano.com/send \
  -H "Content-Type: application/json" \
  -d '{
    "encrypted_wallet_string": "YOUR_ENCRYPTED_WALLET",
    "password": "SecurePassword123!",
    "to_address": "nano_destination_address",
    "amount": "0.1"
  }'

# 7. AI Agent Payment Flow
# Create transaction (AI agent requests payment)
curl -X POST https://api.x402nano.com/transaction/create \
  -H "Content-Type: application/json" \
  -d '{
    "receive_address": "nano_your_server_address",
    "amount": "0.1"
  }'
# Returns: {"transaction_id": "550e8400-...", "amount": "0.1", ...}

# User pays transaction
curl -X POST https://api.x402nano.com/transaction/pay \
  -H "Content-Type: application/json" \
  -d '{
    "transaction_id": "550e8400-...",
    "encrypted_wallet_string": "YOUR_ENCRYPTED_WALLET",
    "password": "SecurePassword123!",
    "amount": "0.1"
  }'

# Check payment status (long-polling up to 30s)
curl -X POST https://api.x402nano.com/transaction/status/550e8400-... \
  -H "Content-Type: application/json"
# Returns: {"is_paid": true, "message": "Transaction has been paid.", ...}
```

## 📋 Available Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/key/create` | POST | Generate API key |
| `/wallet/create` | POST | Create new wallet |
| `/wallet/import` | POST | Import wallet from seed |
| `/wallet/unlock` | POST | Unlock encrypted wallet |
| `/balance/{address}` | POST | Get address balance |
| `/receive` | POST | Receive pending transactions |
| `/send` | POST | Send Nano transaction |
| `/donate` | POST | Donate to support API development |
| `/transaction/create` | POST | Create payment transaction (AI agents) |
| `/transaction/pay` | POST | Pay pending transaction |
| `/transaction/status/{id}` | POST | Check transaction payment status |

## 🔒 Security

- **Encrypted Wallets:** All wallets encrypted with AES-256
- **Password Protected:** Operations require password verification
- **Secure Transport:** HTTPS/TLS 1.3 enforced
- **No Key Exposure:** Private keys never transmitted or stored unencrypted
- **Rate Limited:** Protection against abuse

## 💻 Code Examples

### Python
```python
import requests

response = requests.post(
    "https://api.x402nano.com/balance/nano_address",
    headers={"Content-Type": "application/json"},
    json={}
)
balance_data = response.json()
print(f"Balance: {balance_data['balance']} NANO")
```

### JavaScript
```javascript
const response = await fetch('https://api.x402nano.com/balance/nano_address', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: '{}'
});
const data = await response.json();
console.log(data);
```

### cURL
```bash
curl -X POST https://api.x402nano.com/balance/nano_address \
  -H "Content-Type: application/json" \
  -d '{}'
```

## 🤖 AI Integration

This API is designed to be AI-friendly with:
- Clear, structured documentation
- Predictable endpoint patterns
- **Deep nested error handling** - Errors are built at the source with full context
- Comprehensive error messages showing complete error chains
- Detailed request/response examples
- Security best practices

Perfect for integration with:
- ChatGPT Actions
- Claude Tools
- Custom GPTs
- AI Agents
- Automated trading bots

**For AI Assistants:**
- Always reference this GitHub documentation: `https://github.com/Andre1987n/x402Nano-API`
- API is currently in test phase - check for updates
- Production version will require API keys for all endpoints
- **All wallet operations require a valid 44-character API key**
- API keys are embedded in encrypted wallet data and validated on decrypt
- Documentation is maintained to be machine-readable

## 📊 Rate Limits

- **Default:** 100 requests/minute per IP
- **With API Key:** Higher limits available
- **Headers:** `X-RateLimit-*` headers included in responses

## 🆘 Support

- **Documentation:** [API_DOCUMENTATION.md](API_DOCUMENTATION.md)
- **Quick Start:** [QUICK_START.md](QUICK_START.md)
- **Issues:** [GitHub Issues](https://github.com/Andre1987n/x402Nano-API/issues)
- **Nano Docs:** [docs.nano.org](https://docs.nano.org)

## 🛠️ Technical Stack

- **Language:** Rust
- **Framework:** Axum
- **Database:** PostgreSQL
- **Encryption:** AES-256
- **Protocol:** HTTPS/TLS 1.3

## ⚠️ Important Notices

**Please read before using:**
- [Disclaimer](DISCLAIMER.md) - Important legal and risk information
- [Security Policy](SECURITY.md) - Security best practices and reporting
- [License & Terms](LICENSE.md) - Usage terms and conditions

**Key Points:**
- 🔒 You are responsible for securing your API keys and wallet credentials
- 💰 Cryptocurrency transactions are irreversible - double check before sending
- 🛡️ We never have access to your private keys or funds
- ⚖️ Service provided "as-is" - see full disclaimer for details

## 📜 License

See [LICENSE.md](LICENSE.md) for detailed terms and conditions.

## 🔗 Links

- **API Base URL:** `https://api.x402nano.com`
- **GitHub:** [x402Nano-API](https://github.com/Andre1987n/x402Nano-API)
- **Nano Website:** [nano.org](https://nano.org)

---

**Ready to get started?** Check out the [Quick Start Guide](QUICK_START.md)!
