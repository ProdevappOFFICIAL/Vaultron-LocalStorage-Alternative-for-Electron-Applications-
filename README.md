# 🔐 Vaultron

**Highly secure, offline-capable vault system for Electron desktop applications**

Vaultron is a production-grade TypeScript NPM package that provides extremely secure local storage for sensitive data in Electron applications. It's designed to replace localStorage with military-grade security features including tamper resistance, hardware binding, decoy strategies, and self-destruction capabilities.

## ✨ Features

- **🔒 Military-Grade Encryption**: AES-256-GCM encryption with Argon2id key derivation
- **🛡️ Tamper Detection**: SHA-256 checksums with automatic violation detection
- **💻 Hardware Binding**: Vault tied to specific machine fingerprints
- **🎭 Decoy Generation**: Creates fake vault files to confuse attackers
- **💥 Self-Destruction**: Automatic secure wipe on security violations
- **🌙 Offline-First**: No network dependencies, fully local operation
- **⚡ localStorage-like API**: Familiar interface for easy adoption
- **🔧 Development/Production Modes**: Different security levels for different environments

## 📦 Installation

```bash
npm install vaultron
```

**Requirements:**
- Node.js 16+
- Electron 20+
- TypeScript (recommended)

## 🚀 Quick Start

### 1. Create Configuration

Create a `vaultron.config.js` file in your project root:

```javascript
module.exports = {
  passphrase: "your-super-secure-passphrase-here",
  mode: "development", // or "production"
  enableSelfDestruct: false, // true in production
  decoyCount: 5,
  hardwareBound: true,
  vaultPath: "./.vaultron", // only used in development
  destroyOnAccessViolation: true
};
```

### 2. Initialize and Use

```typescript
import Vaultron from 'vaultron';

async function initApp() {
  try {
    // Initialize the vault (call once at app startup)
    await Vaultron.init();
    
    // Use like localStorage
    Vaultron.setItem('user-token', 'abc123');
    Vaultron.setItem('api-key', 'secret-key-here');
    
    // Retrieve data
    const token = Vaultron.getItem('user-token');
    console.log('Token:', token);
    
    // Remove items
    Vaultron.removeItem('user-token');
    
    // Clear all data
    Vaultron.clear();
    
    // Check vault status
    console.log('Status:', Vaultron.getStatus()); // 'READY', 'LOCKED', or 'DESTROYED'
    
    // Setup destruction callback
    Vaultron.onDestruct(() => {
      console.log('Vault was destroyed due to security violation!');
      // Handle cleanup, show warning to user, etc.
    });
    
  } catch (error) {
    console.error('Failed to initialize vault:', error);
  }
}

initApp();
```

## 🔧 Configuration Options

| Option | Type | Description |
|--------|------|-------------|
| `passphrase` | `string` | **Required**. Master passphrase for encryption (use a strong one!) |
| `mode` | `'development' \| 'production'` | **Required**. Affects security strictness and file locations |
| `enableSelfDestruct` | `boolean` | **Required**. Whether to actually destroy files on violations |
| `decoyCount` | `number` | **Required**. Number of fake vault files to generate (0-10 recommended) |
| `hardwareBound` | `boolean` | **Required**. Bind vault to current machine's hardware fingerprint |
| `vaultPath` | `string?` | Custom vault location (development mode only) |
| `destroyOnAccessViolation` | `boolean` | **Required**. Auto-destroy on tamper detection |

### Development vs Production Mode

**Development Mode:**
- Vault stored in project directory or custom `vaultPath`
- Self-destruct is simulated (files not actually deleted unless `enableSelfDestruct: true`)
- Verbose logging enabled
- Less strict security checks
- Easier debugging and inspection

**Production Mode:**
- Vault stored in OS-protected system directories:
  - **Windows**: `%APPDATA%/.system-vault/.vaultron`
  - **macOS**: `~/Library/.vaultron-secure-store`
  - **Linux**: `~/.config/.vaultron/db_secure`
- Decoy files automatically generated
- Debugger detection enabled
- Strict security enforcement
- Silent operation

## 📁 Vault File Structure

When initialized, Vaultron creates this file structure:

```
.vaultron/ (or OS-specific location in production)
├── .vault-db              # AES-GCM encrypted data file
├── .vault-meta            # Encrypted metadata (hardware ID, config hash, etc.)
├── checksums.json         # SHA-256 hashes for tamper detection
└── fake/                  # Decoy files folder
    ├── temp12.log
    ├── debug.sqlite
    ├── wallet-backup.fake
    └── ... (more decoys)
```

## 🛡️ Security Features

### Encryption
- **Algorithm**: AES-256-GCM (authenticated encryption)
- **Key Derivation**: Argon2id with configurable parameters
- **IV**: Randomly generated for each encryption operation
- **Authentication**: Built-in authentication tags prevent tampering

### Tamper Detection
- SHA-256 checksums of all vault files
- Automatic verification on every access
- Immediate destruction on checksum mismatch
- File modification time monitoring

### Hardware Binding
- Machine fingerprinting using CPU, MAC address, and system UUID
- Vault unusable if moved to different hardware
- Prevents vault copying/migration attacks
- Can be disabled for testing environments

### Decoy Strategy
- Generates realistic fake files (SQLite headers, log files, etc.)
- Makes it harder for attackers to identify real vault
- Configurable number of decoys (recommended: 3-7)
- Decoys are indistinguishable from real vault files

### Self-Destruction
- Secure overwrite with random data before deletion
- Triggered by various security violations:
  - Checksum mismatches
  - Hardware fingerprint changes  
  - Debugger detection (production mode)
  - Invalid configuration
- Configurable destruction callbacks for cleanup

## 📚 API Reference

### Core Methods

```typescript
// Initialize vault (required before use)
await Vaultron.init(): Promise<void>

// localStorage-compatible methods
Vaultron.setItem(key: string, value: string): void
Vaultron.getItem(key: string): string | null
Vaultron.removeItem(key: string): void
Vaultron.clear(): void

// Status and lifecycle
Vaultron.getStatus(): 'LOCKED' | 'READY' | 'DESTROYED'
Vaultron.onDestruct(callback: () => void): void
```

### Status Values

- **`LOCKED`**: Vault not yet initialized
- **`READY`**: Vault initialized and ready for operations
- **`DESTROYED`**: Vault destroyed due to security violation

## ⚠️ Security Warnings

### Critical Security Considerations

1. **Passphrase Security**: Use a strong, unique passphrase. This is your primary line of defense.

2. **Production Mode**: Always use `mode: 'production'` in deployed applications.

3. **Self-Destruct**: Set `enableSelfDestruct: true` in production. Test thoroughly in development first.

4. **Hardware Binding**: Be careful with `hardwareBound: true` in development environments with changing hardware.

5. **Electron Only**: Vaultron will throw an error if used outside Electron applications.

### What Vaultron Protects Against

✅ **Data at rest encryption**  
✅ **File tampering detection**  
✅ **Vault copying to other machines**  
✅ **Casual inspection of vault files**  
✅ **Simple reverse engineering attempts**  

### What Vaultron Cannot Protect Against

❌ **Memory dumps while vault is decrypted**  
❌ **Keyloggers capturing the passphrase**  
❌ **Social engineering attacks**  
❌ **Physical hardware compromise**  
❌ **Advanced persistent threats with system-level access**  

## 🔧 Development

### Building from Source

```bash
git clone https://github.com/your-org/vaultron.git
cd vaultron
npm install
npm run build
```

### Running Tests

```bash
npm test
```

### Development Workflow

1. Set `mode: 'development'` in your config
2. Set `enableSelfDestruct: false` for safety
3. Use `vaultPath: './dev-vault'` for easy inspection
4. Enable logging with `NODE_ENV=development`

## 🚨 Troubleshooting

### Common Issues

**Error: "vaultron.config.js not found"**
- Create the configuration file in your project root
- Ensure it exports a valid configuration object

**Error: "Vault is not ready. Status: DESTROYED"**
- Vault was destroyed due to security violation
- Check logs for specific violation reason
- Remove vault directory and reinitialize with correct config

**Error: "Hardware fingerprint mismatch"**
- Machine hardware changed or vault moved to different computer
- Set `hardwareBound: false` if intentional
- Reinitialize vault on new hardware (data will be lost)

**Performance Issues**
- Reduce `decoyCount` if startup is slow
- Consider faster storage (SSD) for vault location
- Check antivirus software isn't scanning vault files

### Debug Mode

Enable verbose logging:

```bash
NODE_ENV=development node your-app.js
```

### Recovery

If vault is corrupted or destroyed:

1. Remove the entire vault directory
2. Call `Vaultron.init()` again
3. Vault will be recreated (previous data lost)
4. Check your configuration for issues

## 📄 License

MIT License - see LICENSE file for details.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests for new functionality
5. Submit a pull request

## 🔮 Roadmap

### Upcoming Features

- **CLI Tools**: Command-line vault inspection and recovery tools
- **GUI Vault Viewer**: Electron-based vault management interface
- **Plugin System**: Custom hooks for `onTamper`, `onDestruct`, `onUnlock` events
- **Auto-Backup**: Time-based recovery and backup strategies
- **2FA Integration**: OTP/TOTP unlock requirements
- **Multi-Vault**: Support for multiple isolated vaults
- **Audit Logging**: Detailed access and modification logs

### Performance Improvements

- **Lazy Loading**: Load vault data on-demand
- **Compression**: Optional data compression for large vaults
- **Caching**: Intelligent in-memory caching strategies
- **Background Sync**: Asynchronous file operations

## 📞 Support

- **Documentation**: Check this README and inline code comments
- **Issues**: Report bugs on GitHub Issues
- **Security**: Report security vulnerabilities privately to security@yourorg.com
- **Community**: Join discussions in GitHub Discussions

---

**⚠️ Important**: Vaultron is a security tool, but it's not magic. Always follow security best practices, keep your dependencies updated, and consider professional security audits for critical applications.

**🔒 Remember**: The security of your data ultimately depends on the security of your entire system, including your Electron application, the user's machine, and operational security practices.
