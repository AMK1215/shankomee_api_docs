# Launch Game API Documentation

## Overview

The Launch Game API allows client sites to launch Shan Komee games for their players. This endpoint provides secure game launching functionality with signature verification and automatic player management.

## Base URL

```
Production: https://luckymillion.pro/api/client/launch-game
```

## Authentication

This endpoint uses **signature-based authentication** instead of traditional API keys. All requests must include a valid signature generated using your secret key.

## Endpoint

```http
POST /api/client/launch-game
```

## Request Parameters

| Parameter | Type | Required | Description | Example |
|-----------|------|----------|-------------|---------|
| `agent_code` | string | Yes | Your unique agent code | "C902" |
| `product_code` | integer | Yes | Product code for Shan games | 100200 |
| `game_type` | string | Yes | Type of game | "Shan" |
| `member_account` | string | Yes | Player's unique account identifier | "PLAYER0101" |
| `balance` | decimal | Yes | Player's current balance | 1000.00 |
| `request_time` | integer | Yes | Unix timestamp of the request | 1757381386 |
| `sign` | string | Yes | MD5 signature for request verification | "acc4183f7b697f0370b34c3a94e91ab8" |
| `nickname` | string | No | Optional player nickname | "TestPlayer" |
| `callback_url` | string | No | URL for game callbacks | "https://example.com/callback" |

## Signature Generation

The signature is generated using the following formula:

```
sign = md5(request_time + secret_key + 'launchgame' + agent_code)
```

### Signature Generation Examples

#### PHP
```php
<?php
function generateSignature($requestTime, $secretKey, $agentCode) {
    $signString = $requestTime . $secretKey . 'launchgame' . $agentCode;
    return md5($signString);
}

// Example usage
$requestTime = time();
$secretKey = 'your_secret_key_here';
$agentCode = 'C902';
$signature = generateSignature($requestTime, $secretKey, $agentCode);
?>
```

#### JavaScript
```javascript
function generateSignature(requestTime, secretKey, agentCode) {
    const crypto = require('crypto');
    const signString = requestTime + secretKey + 'launchgame' + agentCode;
    return crypto.createHash('md5').update(signString).digest('hex');
}

// Example usage
const requestTime = Math.floor(Date.now() / 1000);
const secretKey = 'your_secret_key_here';
const agentCode = 'C902';
const signature = generateSignature(requestTime, secretKey, agentCode);
```

#### Python
```python
import hashlib
import time

def generate_signature(request_time, secret_key, agent_code):
    sign_string = str(request_time) + secret_key + 'launchgame' + agent_code
    return hashlib.md5(sign_string.encode()).hexdigest()

# Example usage
request_time = int(time.time())
secret_key = 'your_secret_key_here'
agent_code = 'C902'
signature = generate_signature(request_time, secret_key, agent_code)
```

## Request Example

```json
{
    "agent_code": "C902",
    "product_code": 100200,
    "game_type": "Shan",
    "member_account": "PLAYER0101",
    "balance": 1000.00,
    "request_time": 1757381386,
    "sign": "acc4183f7b697f0370b34c3a94e91ab8",
    "nickname": "TestPlayer",
    "callback_url": "https://example.com/callback"
}
```

## Response Format

### Success Response

```json
{
    "code": 200,
    "message": "Game launched successfully",
    "url": "https://golden-mm-shan.vercel.app/?user_name=PLAYER0101&balance=1000.00"
}
```

### Error Responses

#### Invalid Signature (401)
```json
{
    "code": 401,
    "message": "Invalid signature"
}
```

#### Validation Error (422)
```json
{
    "code": 422,
    "message": "Validation failed",
    "errors": {
        "agent_code": ["The agent code field is required."],
        "balance": ["The balance must be at least 0."]
    }
}
```

#### Agent Not Found (404)
```json
{
    "code": 404,
    "message": "Agent not found"
}
```

#### Server Error (500)
```json
{
    "code": 500,
    "message": "Internal server error"
}
```

## Integration Examples

### PHP Integration

```php
<?php
class ShanGameLauncher {
    private $baseUrl;
    private $agentCode;
    private $secretKey;
    
    public function __construct($baseUrl, $agentCode, $secretKey) {
        $this->baseUrl = $baseUrl;
        $this->agentCode = $agentCode;
        $this->secretKey = $secretKey;
    }
    
    public function launchGame($memberAccount, $balance, $nickname = null, $callbackUrl = null) {
        $requestTime = time();
        $signature = $this->generateSignature($requestTime);
        
        $data = [
            'agent_code' => $this->agentCode,
            'product_code' => 100200,
            'game_type' => 'Shan',
            'member_account' => $memberAccount,
            'balance' => $balance,
            'request_time' => $requestTime,
            'sign' => $signature
        ];
        
        if ($nickname) {
            $data['nickname'] = $nickname;
        }
        
        if ($callbackUrl) {
            $data['callback_url'] = $callbackUrl;
        }
        
        $response = $this->makeRequest('/client/launch-game', $data);
        
        if ($response && $response['code'] === 200) {
            return $response['url'];
        }
        
        throw new Exception($response['message'] ?? 'Launch failed');
    }
    
    private function generateSignature($requestTime) {
        $signString = $requestTime . $this->secretKey . 'launchgame' . $this->agentCode;
        return md5($signString);
    }
    
    private function makeRequest($endpoint, $data) {
        $url = $this->baseUrl . $endpoint;
        
        $options = [
            'http' => [
                'header' => "Content-type: application/json\r\n",
                'method' => 'POST',
                'content' => json_encode($data)
            ]
        ];
        
        $context = stream_context_create($options);
        $result = file_get_contents($url, false, $context);
        
        if ($result === FALSE) {
            return false;
        }
        
        return json_decode($result, true);
    }
}

// Usage
$launcher = new ShanGameLauncher(
    'https://luckymillion.pro/api',
    'C902',
    'your_secret_key_here'
);

try {
    $gameUrl = $launcher->launchGame('PLAYER0101', 1000.00, 'TestPlayer');
    header("Location: $gameUrl");
    exit;
} catch (Exception $e) {
    echo "Error: " . $e->getMessage();
}
?>
```

### JavaScript Integration

```javascript
class ShanGameLauncher {
    constructor(baseUrl, agentCode, secretKey) {
        this.baseUrl = baseUrl;
        this.agentCode = agentCode;
        this.secretKey = secretKey;
    }
    
    async launchGame(memberAccount, balance, nickname = null, callbackUrl = null) {
        const requestTime = Math.floor(Date.now() / 1000);
        const signature = this.generateSignature(requestTime);
        
        const data = {
            agent_code: this.agentCode,
            product_code: 100200,
            game_type: 'Shan',
            member_account: memberAccount,
            balance: balance,
            request_time: requestTime,
            sign: signature
        };
        
        if (nickname) {
            data.nickname = nickname;
        }
        
        if (callbackUrl) {
            data.callback_url = callbackUrl;
        }
        
        try {
            const response = await fetch(`${this.baseUrl}/client/launch-game`, {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Accept': 'application/json'
                },
                body: JSON.stringify(data)
            });
            
            const result = await response.json();
            
            if (result.code === 200) {
                return result.url;
            }
            
            throw new Error(result.message);
        } catch (error) {
            console.error('Launch game error:', error);
            throw error;
        }
    }
    
    generateSignature(requestTime) {
        const crypto = require('crypto');
        const signString = requestTime + this.secretKey + 'launchgame' + this.agentCode;
        return crypto.createHash('md5').update(signString).digest('hex');
    }
}

// Usage
const launcher = new ShanGameLauncher(
    'https://luckymillion.pro/api',
    'C902',
    'your_secret_key_here'
);

launcher.launchGame('PLAYER0101', 1000.00, 'TestPlayer')
    .then(url => {
        window.location.href = url;
    })
    .catch(error => {
        console.error('Failed to launch game:', error);
        alert('Failed to launch game: ' + error.message);
    });
```

### Python Integration

```python
import requests
import hashlib
import time
import json

class ShanGameLauncher:
    def __init__(self, base_url, agent_code, secret_key):
        self.base_url = base_url
        self.agent_code = agent_code
        self.secret_key = secret_key
    
    def launch_game(self, member_account, balance, nickname=None, callback_url=None):
        request_time = int(time.time())
        signature = self.generate_signature(request_time)
        
        data = {
            'agent_code': self.agent_code,
            'product_code': 100200,
            'game_type': 'Shan',
            'member_account': member_account,
            'balance': balance,
            'request_time': request_time,
            'sign': signature
        }
        
        if nickname:
            data['nickname'] = nickname
        
        if callback_url:
            data['callback_url'] = callback_url
        
        try:
            response = requests.post(
                f"{self.base_url}/client/launch-game",
                json=data,
                headers={'Content-Type': 'application/json'}
            )
            
            result = response.json()
            
            if result.get('code') == 200:
                return result['url']
            
            raise Exception(result.get('message', 'Launch failed'))
            
        except Exception as e:
            print(f"Launch game error: {e}")
            raise e
    
    def generate_signature(self, request_time):
        sign_string = str(request_time) + self.secret_key + 'launchgame' + self.agent_code
        return hashlib.md5(sign_string.encode()).hexdigest()

# Usage
launcher = ShanGameLauncher(
    'https://luckymillion.pro/api',
    'C902',
    'your_secret_key_here'
)

try:
    game_url = launcher.launch_game('PLAYER0101', 1000.00, 'TestPlayer')
    print(f"Game URL: {game_url}")
except Exception as e:
    print(f"Error: {e}")
```

### cURL Example

```bash
#!/bin/bash

# Configuration
BASE_URL="https://luckymillion.pro/api"
AGENT_CODE="C902"
SECRET_KEY="your_secret_key_here"
MEMBER_ACCOUNT="PLAYER0101"
BALANCE="1000.00"
NICKNAME="TestPlayer"

# Generate signature
REQUEST_TIME=$(date +%s)
SIGN_STRING="${REQUEST_TIME}${SECRET_KEY}launchgame${AGENT_CODE}"
SIGNATURE=$(echo -n "$SIGN_STRING" | md5sum | cut -d' ' -f1)

# Make request
curl -X POST "${BASE_URL}/client/launch-game" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d "{
    \"agent_code\": \"${AGENT_CODE}\",
    \"product_code\": 100200,
    \"game_type\": \"Shan\",
    \"member_account\": \"${MEMBER_ACCOUNT}\",
    \"balance\": ${BALANCE},
    \"request_time\": ${REQUEST_TIME},
    \"sign\": \"${SIGNATURE}\",
    \"nickname\": \"${NICKNAME}\"
  }"
```

## Player Management

The API automatically handles player management:

### New Players
- Automatically creates new players in the system
- Sets initial balance as specified in the request
- Links player to the specified agent

### Existing Players
- Updates player balance if different from current balance
- Maintains agent relationship
- Preserves player history and settings

## Game URL Structure

The returned game URL follows this pattern:
```
https://golden-mm-shan.vercel.app/?user_name={member_account}&balance={balance}
```

## Error Handling

### Common Error Scenarios

1. **Invalid Signature**: Check your secret key and signature generation
2. **Agent Not Found**: Verify your agent code is correct
3. **Validation Errors**: Ensure all required fields are provided and valid
4. **Server Errors**: Contact support if persistent

### Error Response Codes

| Code | Description | Action |
|------|-------------|--------|
| 200 | Success | Proceed with game URL |
| 401 | Invalid signature | Check signature generation |
| 404 | Agent not found | Verify agent code |
| 422 | Validation failed | Check request parameters |
| 500 | Server error | Contact support |

## Security Considerations

1. **Secret Key Protection**: Never expose your secret key in client-side code
2. **HTTPS Only**: Always use HTTPS for API communications
3. **Request Time Validation**: The API validates request time to prevent replay attacks
4. **Signature Verification**: Always verify signatures on your side for callbacks
5. **Input Validation**: Validate all input data before sending requests

## Testing

### Test Environment
Use the same endpoint with test credentials for development and testing.

### Test Data
```json
{
    "agent_code": "TEST001",
    "product_code": 100200,
    "game_type": "Shan",
    "member_account": "TEST_PLAYER_001",
    "balance": 100.00,
    "request_time": 1757381386,
    "sign": "generated_signature_here",
    "nickname": "Test Player"
}
```

## Support

For technical support or questions about the Launch Game API:

- **Email**: supervip445@gmail.com
- **Telegram**: @aiworld2048
- **Documentation**: This document and related API docs
- **Test Environment**: Available for development testing

## Changelog

### Version 1.0.0
- Initial release of Launch Game API
- Signature-based authentication
- Automatic player management
- Shan game integration
- Comprehensive error handling

---

*This documentation is updated regularly. Please check for the latest version.*
