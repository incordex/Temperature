# Function Signature
> topic hash for event signatures.

**Take the first few bytes of the Keccak-256 hash of a function signature:**

1. Create the function signature (function name + parameter types)
2. Compute the Keccak-256 hash
3. Extract the first N bytes

**Important notes:**

- Function selectors typically use the first 4 bytes (32 bits)
- The signature format is `functionName(param1Type,param2Type)` with no spaces
- Keccak-256 ≠ SHA-256 (they're different algorithms)
- No spaces in function signatures: `transfer(address,uint256)` not `transfer(address, uint256)`
- Parameter types must match exactly (e.g., uint256 not uint)

```javascript
const { keccak256 } = require('js-sha3');

function getFunctionSelectorSha3(functionSignature, bytesToTake = 4)
{
  // Compute Keccak-256 hash of the function signature
  const hash = keccak256(functionSignature);

  // Take the first N bytes (default 4 bytes for Ethereum function selectors)
  // Remove '0x' prefix, then take first N bytes (each byte = 2 hex chars)
  const selector = '0x' + hash.slice(0, bytesToTake * 2);
  
  return selector;
}

console.log('transfer(address,uint256):', getFunctionSelectorSha3('transfer(address,uint256)'));

```

## Browser-compatible (Web Crypto API implementation)

```javascript
async function getSHA256Selector(functionSignature, bytesToTake = 4)
{
  const encoder = new TextEncoder();
  const data = encoder.encode(functionSignature);
  
  // This uses SHA-256, NOT Keccak-256 (different algorithm!)
  const hashBuffer = await crypto.subtle.digest('SHA-256', data);
  const hashArray = new Uint8Array(hashBuffer);
  
  // Convert first N bytes to hex
  const firstBytes = hashArray.slice(0, bytesToTake);
  const hex = '0x' + Array.from(firstBytes)
    .map(b => b.toString(16).padStart(2, '0'))
    .join('');
  
  return hex;
}

// getSHA256Selector('transfer(address,uint256)').then(console.log);

// ============================================================================
// Utility function to create function signatures from ABI
function createFunctionSignature(functionName, paramTypes) {
  return `${functionName}(${paramTypes.join(',')})`;
}

// Example usage
const signature = createFunctionSignature('transferFrom', ['address', 'address', 'uint256']);
console.log('\nGenerated signature:', signature);
console.log('Selector:', getFunctionSelector(signature));
```
