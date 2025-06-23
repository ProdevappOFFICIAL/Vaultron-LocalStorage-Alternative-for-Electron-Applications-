# Vaultron Complete Guide

## 📋 Table of Contents
1. [Installation](#installation)
2. [How It Works](#how-it-works)
3. [Folder Structure & Storage](#folder-structure--storage)
4. [Configuration](#configuration)
5. [Basic Usage](#basic-usage)
6. [Advanced Features](#advanced-features)
7. [Security Features](#security-features)
8. [Troubleshooting](#troubleshooting)

---

## 🚀 Installation

### Step 1: Install the Package
```bash
npm install vaultron
# or
yarn add vaultron
```

### Step 2: Verify Electron Environment
Vaultron **only works in Electron applications**. It will throw an error if used in:
- Web browsers
- Node.js servers
- React/Vue web apps

---

## 🔧 How It Works

### Architecture Overview
```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Your App      │───▶│    Vaultron      │───▶│  Encrypted      │
│                 │    │                  │    │  Vault Files    │
│ setItem("key")  │    │ ✓ Encrypt        │    │                 │
│ getItem("key")  │    │ ✓ Compress       │    │ vault.bin       │
│                 │    │ ✓ Obfuscate      │    │ decoy_1.bin     │
└─────────────────┘    └──────────────────┘    │ decoy_2.bin     │
                                               └─────────────────┘
```

### Data Flow
1. **Input**: Your data (string or object)
2. **Processing**: JSON stringify → AES-256 encrypt → Compress → XOR obfuscate
3. **Storage**: Write to encrypted binary file
4. **Retrieval**: Reverse process with integrity verification

---

## 📁 Folder Structure & Storage

### Development Mode
When `mode: 'development'` in config:
```
your-electron-project/
├── src/
├── package.json
├── vaultron.config.js          ← Your config file
└── .vaultron/                  ← Created automatically
    └── default/                ← Vault name folder
        ├── vault.bin           ← Main encrypted vault
        ├── decoy_1.bin         ← Fake files for security
        ├── decoy_2.bin
        └── decoy_3.bin
```

### Production Mode
When `mode: 'production'` in config:

**Windows:**
```
C:\Users\YourName\AppData\Roaming\vaultron\
└── your-vault-name/
    ├── vault.bin
    └── decoy_*.bin
```

**macOS:**
```
/Users/YourName/Library/Application Support/vaultron/
└── your-vault-name/
    ├── vault.bin
    └── decoy_*.bin
```

**Linux:**
```
/home/yourname/.config/vaultron/
└── your-vault-name/
    ├── vault.bin
    └── decoy_*.bin
```

---

## ⚙️ Configuration

### Option 1: Zero Configuration (Quick Start)
```javascript
import vaultron from 'vaultron';

// Uses default settings
await vaultron.init();
await vaultron.setItem('myKey', 'myValue');
```

**Default Settings:**
- Mode: `development`
- Passphrase: `'default-passphrase-change-me'`
- Vault name: `'default'`
- Security features: All enabled

### Option 2: Config File (Recommended)

Create `vaultron.config.js` in your project root:

```javascript
// vaultron.config.js
module.exports = {
  // Required
  mode: 'production',                    // 'development' or 'production'
  passphrase: 'your-super-secure-key',   // String or function
  
  // Optional Security
  selfDestruct: true,        // Auto-wipe on tamper detection
  decoyCount: 10,           // Number of fake files (0-50)
  vaultName: 'myapp',       // Custom vault identifier
  osLock: true,             // Lock to current machine
  
  // Optional Features
  auditLogs: true,          // Track all operations
  compressionEnabled: true, // Compress data
  obfuscationEnabled: true  // Extra XOR layer
};
```

### Option 3: Dynamic Configuration
```javascript
// For environment-based config
module.exports = {
  mode: process.env.NODE_ENV === 'production' ? 'production' : 'development',
  passphrase: () => process.env.VAULT_PASSPHRASE || 'fallback-key',
  selfDestruct: process.env.NODE_ENV === 'production',
  vaultName: process.env.APP_NAME || 'myapp'
};
```

### Option 4: Custom Config Path
```javascript
// Load config from custom location
await vaultron.init('./config/my-vault-config.js');
```

---

## 🎯 Basic Usage

### Complete Example
```javascript
// main.js (Electron main process)
import vaultron from 'vaultron';

async function setupVault() {
  try {
    // Initialize (loads config automatically)
    await vaultron.init();
    console.log('✅ Vaultron initialized');
    
    // Store different data types
    await vaultron.setItem('apiKey', 'sk-1234567890abcdef');
    await vaultron.setItem('userSettings', {
      theme: 'dark',
      language: 'en',
      notifications: true,
      lastLogin: new Date().toISOString()
    });
    await vaultron.setItem('secretToken', 'very-secret-jwt-token');
    
    // Retrieve data
    const apiKey = await vaultron.getItem('apiKey');
    const settings = await vaultron.getItem('userSettings');
    
    console.log('API Key:', apiKey);
    console.log('User Theme:', settings.theme);
    
    // List all stored keys
    const allKeys = vaultron.listKeys();
    console.log('Stored keys:', allKeys);
    // Output: ['apiKey', 'userSettings', 'secretToken']
    
    // Remove item
    await vaultron.removeItem('secretToken');
    
  } catch (error) {
    console.error('Vault error:', error.message);
  }
}

// Call when app is ready
app.whenReady().then(setupVault);
```

### Renderer Process Usage
```javascript
// preload.js (if using contextBridge)
const { contextBridge } = require('electron');
const vaultron = require('vaultron');

contextBridge.exposeInMainWorld('vault', {
  init: () => vaultron.init(),
  set: (key, value) => vaultron.setItem(key, value),
  get: (key) => vaultron.getItem(key),
  remove: (key) => vaultron.removeItem(key),
  listKeys: () => vaultron.listKeys()
});

// renderer.js
await window.vault.init();
await window.vault.set('userPref', { theme: 'dark' });
const pref = await window.vault.get('userPref');
```

---

## 🔐 Advanced Features

### 1. Self-Destruct Mode
```javascript
// Enable in config
module.exports = {
  selfDestruct: true  // Wipes vault on tamper detection
};

// Manual trigger (USE WITH EXTREME CAUTION!)
await vaultron.selfDestruct();
// This will:
// 1. Overwrite vault file with random data 3x
// 2. Overwrite with zeros
// 3. Delete all files
// 4. Clear memory
// 5. Remove directory
```

### 2. Audit Logging
```javascript
// Get operation history
const auditLog = vaultron.getAuditLog();
console.log(auditLog);
/* Output:
[
  {
    action: 'init',
    timestamp: 1704067200000,
    machineId: 'abc123...'
  },
  {
    action: 'write',
    key: 'apiKey',
    timestamp: 1704067201000,
    machineId: 'abc123...'
  }
]
*/
```

### 3. Vault Metadata
```javascript
const metadata = vaultron.getMetadata();
console.log(metadata);
/* Output:
{
  version: '1.0.0',
  created: 1704067200000,
  lastModified: 1704067300000,
  machineId: 'abc123...',
  osInfo: 'win32-10.0.19045'
}
*/
```

### 4. Plugin System
```javascript
// Create a custom plugin
const myPlugin = {
  name: 'AccessLogger',
  beforeRead: (key) => {
    console.log(`Reading key: ${key}`);
    return true; // Allow read
  },
  afterWrite: (key, value) => {
    console.log(`Wrote key: ${key}`);
  },
  beforeDelete: (key) => {
    if (key === 'critical-data') {
      console.log('Blocked deletion of critical data');
      return false; // Block deletion
    }
    return true;
  }
};

// Register plugin
vaultron.registerPlugin(myPlugin);
```

### 5. Export for Vault Viewer
```javascript
// Export vault data (for external tools)
const exportedData = await vaultron.exportVault('export-password');
console.log(exportedData); // Base64 encoded encrypted data

// Save to file for Vault Viewer app
const fs = require('fs');
fs.writeFileSync('vault-export.vaultron', exportedData);
```

---

## 🛡️ Security Features

### Machine Locking
```javascript
// When osLock: true, vault is tied to:
// - Machine ID (MAC address based)
// - OS version
// - Hardware fingerprint

// If vault is moved to different machine:
// → Throws error or self-destructs
```

### Tamper Detection
```javascript
// Vault detects:
// ✓ File corruption
// ✓ Manual file editing
// ✓ Checksum mismatches
// ✓ Unauthorized machine access

// Response (if selfDestruct: true):
// → Immediate vault wipe
// → Audit log entry
// → Error thrown
```

### Encryption Layers
```
Your Data
    ↓
JSON.stringify()
    ↓
AES-256-GCM Encryption (with Argon2 key)
    ↓
Zlib Compression
    ↓
XOR Obfuscation
    ↓
Binary File (.vault)
```

### Decoy Files
```javascript
// Generates fake files to confuse attackers
decoy_1.bin  // Random data
decoy_2.bin  // Random data
decoy_3.bin  // Random data
vault.bin    // Real encrypted data (hidden among decoys)
```

---

## 🔍 Troubleshooting

### Common Issues

#### 1. "Vaultron can only be used within Electron applications"
```javascript
// ❌ Problem: Running in browser or Node.js
// ✅ Solution: Only use in Electron main/renderer process

// Check if properly in Electron:
console.log('Electron version:', process.versions.electron);
```

#### 2. "Vaultron not initialized"
```javascript
// ❌ Problem: Using before init()
await vaultron.setItem('key', 'value'); // Error!

// ✅ Solution: Always init first
await vaultron.init();
await vaultron.setItem('key', 'value'); // Works!
```

#### 3. "Vault locked to different machine"
```javascript
// ❌ Problem: Moved vault to different computer with osLock: true
// ✅ Solutions:
// Option 1: Set osLock: false in config
// Option 2: Export/import vault data
// Option 3: Use selfDestruct and recreate
```

#### 4. "Failed to load config file"
```javascript
// ❌ Problem: Syntax error in vaultron.config.js
// ✅ Solution: Check config syntax
module.exports = {
  mode: 'development',  // ← Comma needed
  passphrase: 'test'    // ← No comma on last item
};
```

#### 5. Permission Errors
```bash
# Windows: Run as administrator if needed
# macOS/Linux: Check folder permissions
chmod 755 ~/.config/vaultron
```

### Debug Mode
```javascript
// Add to config for verbose logging
module.exports = {
  mode: 'development',
  passphrase: 'test',
  debug: true  // Enable debug output
};
```

### Recovery Options
```javascript
// If vault is corrupted:
// 1. Check if backup exists
// 2. Try export/import
// 3. Last resort: selfDestruct and recreate

// Safe recovery:
try {
  await vaultron.init();
} catch (error) {
  console.log('Vault corrupted, creating new one...');
  await vaultron.selfDestruct();
  await vaultron.init();
}
```

---

## 📊 Performance Considerations

### File Sizes
- Empty vault: ~1KB
- 100 entries: ~10-50KB (depending on data)
- 1000 entries: ~100-500KB
- Decoy files: 1-3KB each

### Operations Speed
- `setItem()`: ~1-5ms
- `getItem()`: ~1-3ms  
- `init()`: ~10-50ms (first time)
- `selfDestruct()`: ~100-500ms

### Memory Usage
- Runtime: ~1-5MB
- Encryption overhead: ~2x data size during operations

---

## 🎉 Best Practices

1. **Always use strong passphrases in production**
2. **Enable selfDestruct for sensitive applications**
3. **Use environment variables for passphrases**
4. **Regular audit log reviews**
5. **Test vault recovery procedures**
6. **Keep decoy count reasonable (5-15)**
7. **Use meaningful vault names for multiple apps**

---

## 📞 Support

For issues or questions:
1. Check this guide first
2. Review error messages carefully
3. Enable debug mode for detailed logs
4. Check Electron version compatibility
5. Verify file permissions

Remember: Vaultron prioritizes security over convenience. Some restrictions are intentional security features!
