# Webhook Integration Guide

**Version:** 0.2.0  
**Last Updated:** January 10, 2026

---

## Overview

Webhooks provide instant push notifications when transactions are paid. Instead of repeatedly polling the `/transaction/status` endpoint, your server receives an HTTP POST request the moment payment is confirmed.

**Benefits:**
- ⚡ **Instant notifications** - Payment confirmed in < 1 second
- 📉 **Reduced server load** - No need for polling loops
- 🤖 **AI-friendly** - Perfect for autonomous agents and automation
- 🔄 **Automatic retry** - We handle delivery failures automatically

---

## Quick Start

### 1. Create Transaction with Webhook

```bash
curl -X POST https://api.x402nano.com/transaction/create \
  -H "Content-Type: application/json" \
  -d '{
    "receive_address": "nano_1your_server_address",
    "amount": "1.5",
    "webhook_url": "https://yoursite.com/webhook",
    "metadata": "{\"order_id\":\"ORD-123\",\"customer\":\"user@example.com\"}"
  }'
```

**Response:**
```json
{
  "receive_address": "nano_1your_server_address",
  "amount": "1.5",
  "transaction_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

### 2. Implement Webhook Endpoint

Your server must respond to POST requests at the `webhook_url`:

```python
from flask import Flask, request
import json

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def handle_webhook():
    # Get headers
    transaction_id = request.headers.get('X-Transaction-Id', '')
    
    # Get payload
    data = request.json
    
    # Process payment
    print(f"Payment received!")
    print(f"Transaction ID: {data['transaction_id']}")
    print(f"Amount: {data['amount']} Nano")
    print(f"Address: {data['receive_address']}")
    
    # Parse metadata
    if data.get('metadata'):
        metadata = json.loads(data['metadata'])
        order_id = metadata.get('order_id')
        print(f"Order ID: {order_id}")
        # Mark order as paid, start shipping, etc.
    
    # IMPORTANT: Return 200 OK
    return 'OK', 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### 3. Whitelist Our Server IP (Highly Recommended)

**Our Server IP:** `57.129.16.77`

**Nginx Example:**
```nginx
location /webhook {
    # Only allow x402 Nano server
    allow 57.129.16.77;
    deny all;
    
    proxy_pass http://localhost:5000;
}
```

**Apache Example:**
```apache
<Location "/webhook">
    Order deny,allow
    Deny from all
    Allow from 57.129.16.77
</Location>
```

**Firewall (iptables):**
```bash
# Allow webhook from x402 server only
iptables -A INPUT -p tcp --dport 5000 -s 57.129.16.77 -j ACCEPT
iptables -A INPUT -p tcp --dport 5000 -j DROP
```

---

## Webhook Payload

### Headers

```
Content-Type: application/json
X-Transaction-Id: 550e8400-e29b-41d4-a716-446655440000
```

### Body

```json
{
  "transaction_id": "550e8400-e29b-41d4-a716-446655440000",
  "receive_address": "nano_1your_server_address",
  "amount": "1.5",
  "metadata": "{\"order_id\":\"ORD-123\",\"customer\":\"user@example.com\"}",
  "paid_at": "2026-01-10T14:30:00Z"
}
```

**Field Descriptions:**
- `transaction_id` - UUID of the transaction
- `receive_address` - Your Nano address that received payment
- `amount` - Amount paid in Nano (string)
- `metadata` - JSON string with custom data (if provided during creation)
- `paid_at` - ISO 8601 timestamp when payment was confirmed

---

## Security

### IP Whitelisting (Highly Recommended)

**This is the best and simplest method.**

All webhooks originate from our server IP: **57.129.16.77**

Configure your firewall or web server to **only accept webhook requests from this IP address**. This prevents anyone else from sending fake webhook notifications to your endpoint.

**Why it's the best:**
- ✅ Impossible to spoof (requires controlling our server)
- ✅ Simple to implement (one firewall rule)
- ✅ No secret management needed
- ✅ Works at network level (fastest)
- ✅ Industry standard for webhook security

**Implementation:** See the nginx/Apache/firewall examples in the "Whitelist Our Server IP" section above.

---

## Retry Logic

If webhook delivery fails, we automatically retry:

| Attempt | Delay | Total Time |
|---------|-------|------------|
| 1       | 0s    | Immediate  |
| 2       | 30s   | 30 seconds |
| 3       | 5min  | 5 minutes 30 seconds |

**Success Criteria:**
- HTTP status code: 200-299
- Response within 10 seconds

**Logged Information:**
- Each delivery attempt
- HTTP status codes
- Response bodies (truncated)
- Error messages
- Final delivery status

**Note:** After 3 failed attempts, delivery stops. The payment is still completed, but you won't receive the webhook notification. Check transaction status manually if needed.

---

## Best Practices

### 1. Return 200 OK Quickly

Process webhooks asynchronously. Respond with `200 OK` immediately, then process in background:

```python
@app.route('/webhook', methods=['POST'])
def webhook():
    data = request.json
    
    # Queue for background processing
    payment_queue.enqueue(process_payment, data)
    
    # Return immediately
    return 'OK', 200

def process_payment(data):
    # Heavy processing here
    update_database(data)
    send_confirmation_email(data)
    update_inventory(data)
```

### 2. Make Endpoint Idempotent

The same webhook may be delivered multiple times (e.g., if timeout occurs). Handle duplicates gracefully:

```python
def process_payment(data):
    tx_id = data['transaction_id']
    
    # Check if already processed
    if database.transaction_exists(tx_id):
        print(f"Duplicate webhook for {tx_id}, ignoring")
        return
    
    # Process and mark as complete
    database.mark_transaction_paid(tx_id)
    # ... rest of processing
```

### 3. Validate Transaction Data

Always verify the transaction matches your expectations:

```python
def process_payment(data):
    # Parse metadata
    metadata = json.loads(data['metadata'])
    order = database.get_order(metadata['order_id'])
    
    # Verify amount matches
    if float(data['amount']) != float(order.amount):
        logger.error(f"Amount mismatch! Expected {order.amount}, got {data['amount']}")
        # Handle error...
        return
    
    # Verify address
    if data['receive_address'] != os.getenv('NANO_ADDRESS'):
        logger.error(f"Wrong address! Got {data['receive_address']}")
        return
    
    # All good, mark as paid
    order.mark_as_paid()
```

### 4. Log Everything

Keep detailed logs for debugging:

```python
import logging

logger = logging.getLogger(__name__)

@app.route('/webhook', methods=['POST'])
def webhook():
    logger.info(f"Webhook received: {request.headers}")
    logger.info(f"Payload: {request.get_data()}")
    
    try:
        data = request.json
        logger.info(f"Processing transaction {data['transaction_id']}")
        
        # Process...
        
        logger.info(f"Successfully processed {data['transaction_id']}")
        return 'OK', 200
        
    except Exception as e:
        logger.error(f"Webhook processing error: {str(e)}", exc_info=True)
        return 'Error', 500
```

### 5. Monitor Webhook Health

Track webhook delivery success rate:

```python
from prometheus_client import Counter, Histogram

webhook_success = Counter('webhook_success_total', 'Successful webhooks')
webhook_failure = Counter('webhook_failure_total', 'Failed webhooks')
webhook_duration = Histogram('webhook_duration_seconds', 'Webhook processing time')

@app.route('/webhook', methods=['POST'])
def webhook():
    start = time.time()
    
    try:
        # Process webhook...
        webhook_success.inc()
        return 'OK', 200
        
    except Exception as e:
        webhook_failure.inc()
        return 'Error', 500
        
    finally:
        duration = time.time() - start
        webhook_duration.observe(duration)
```

---

## Testing

### Local Testing with ngrok

1. **Start your local server:**
```bash
python webhook_server.py
# Server running on http://localhost:5000
```

2. **Expose with ngrok:**
```bash
ngrok http 5000
# Forwarding: https://abc123.ngrok.io -> http://localhost:5000
```

3. **Create transaction with ngrok URL:**
```bash
curl -X POST https://api.x402nano.com/transaction/create \
  -H "Content-Type: application/json" \
  -d '{
    "receive_address": "nano_...",
    "amount": "0.01",
    "webhook_url": "https://abc123.ngrok.io/webhook"
  }'
```

4. **Pay the transaction and watch your console!**

### Testing with webhook.site

For quick testing without running a server:

1. Go to https://webhook.site
2. Copy your unique URL (e.g., `https://webhook.site/abc-123`)
3. Create transaction with that URL as `webhook_url`
4. Pay the transaction
5. View webhook payload in webhook.site interface

---

## Troubleshooting

### Webhook Not Received

**Check firewall rules:**
```bash
# Test if port is accessible from our server
telnet your-server.com 443
```

**Check server logs:**
```bash
tail -f /var/log/nginx/error.log
tail -f /var/log/your-app/webhook.log
```

**Common issues:**
- Firewall blocking port 443/80
- SSL certificate expired
- Endpoint returns non-200 status code
- Timeout (> 10 seconds to respond)
- Wrong URL (typo in webhook_url)

### Multiple Webhooks Received

This is normal! Possible causes:
- Your server returned 500 error first time (we retried)
- Your server took > 10 seconds to respond (timeout + retry)
- Network issue during first attempt

**Solution:** Make your endpoint idempotent (check `transaction_id` before processing).

---

## Production Checklist

Before going live, ensure:

- [ ] Webhook endpoint is HTTPS (not HTTP)
- [ ] IP whitelisting configured (57.129.16.77 only)
- [ ] Endpoint returns 200 OK within 10 seconds
- [ ] Idempotent handling (duplicate webhooks work fine)
- [ ] Amount/address validation implemented
- [ ] Error logging enabled
- [ ] Monitoring/alerting set up
- [ ] Tested with real payment (small amount)
- [ ] Backup status polling implemented (in case webhook fails)

---

## Example: E-commerce Integration

Complete example for an online store:

```python
from flask import Flask, request
import json
from datetime import datetime

app = Flask(__name__)

@app.route('/webhook', methods=['POST'])
def handle_payment():
    # 1. Get transaction ID
    transaction_id = request.headers.get('X-Transaction-Id', '')
    
    # 2. Parse data
    data = request.json
    transaction_id = data['transaction_id']
    amount = data['amount']
    
    # 3. Extract metadata
    metadata = json.loads(data['metadata'])
    order_id = metadata['order_id']
    customer_email = metadata['customer_email']
    
    # 4. Verify transaction
    order = db.get_order(order_id)
    
    if not order:
        logger.error(f"Order {order_id} not found")
        return 'Order not found', 404
    
    if order.already_paid:
        logger.info(f"Order {order_id} already paid (duplicate webhook)")
        return 'Already processed', 200
    
    if float(amount) != float(order.total):
        logger.error(f"Amount mismatch for {order_id}")
        return 'Amount mismatch', 400
    
    # 5. Mark order as paid
    order.status = 'paid'
    order.paid_at = datetime.utcnow()
    order.nano_transaction_id = transaction_id
    db.save(order)
    
    # 6. Fulfill order
    send_digital_goods(order)
    send_confirmation_email(customer_email, order)
    
    logger.info(f"Order {order_id} successfully paid and fulfilled")
    return 'OK', 200
```

---

## Support

**Issues?** Contact us:
- GitHub Issues: [x402 Nano API Issues](https://github.com/Andre1987n/x402Nano-API/issues)
- Email: support@x402nano.com
- Server IP: 57.129.16.77 (for whitelisting)

**Documentation:**
- [API Reference](API_DOCUMENTATION.md)
- [Quick Start Guide](QUICK_START.md)
- [OpenAPI Schema](openapi-schema.yaml)

---

**Built for AI Agents 🤖 | Zero Fees ⚡ | Instant Settlement 🚀**
