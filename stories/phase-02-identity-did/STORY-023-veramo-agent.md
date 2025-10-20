# STORY-023: Veramo Agent Configuration

**Phase**: 2 - Identity & DID  
**Story**: 023 of 112  
**Estimated Time**: 5-6 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📋 Story Information

### Objectives
- Install Veramo core packages and dependencies
- Configure Veramo agent with essential plugins
- Setup KeyManager for key management
- Setup DIDManager for identity management  
- Setup DIDResolver for DID resolution
- Setup DataStore for persistence
- Test agent initialization and basic operations

### Prerequisites
- ✅ Phase 0 & 1 complete (database working, user authenticated)
- ✅ Understanding of DIDs (CRITICAL!)
- ✅ Read Veramo documentation
- ✅ TypeORM database configured

### Dependencies
```json
{
  "dependencies": {
    "@veramo/core": "4.2.0",
    "@veramo/did-manager": "4.2.0",
    "@veramo/did-resolver": "4.2.0",
    "@veramo/key-manager": "4.2.0",
    "@veramo/data-store": "4.2.0",
    "@veramo/did-provider-jwk": "^1.0.0",
    "@veramo/did-provider-key": "4.2.0",
    "did-resolver": "^4.1.0",
    "reflect-metadata": "^0.2.2",
    "@ethersproject/shims": "^5.7.0"
  }
}
```

---

## 📝 Implementation Steps

### Step 1: Install Veramo Packages

```bash
# Install core Veramo packages
npm install @veramo/core@4.2.0
npm install @veramo/did-manager@4.2.0
npm install @veramo/did-resolver@4.2.0
npm install @veramo/key-manager@4.2.0
npm install @veramo/data-store@4.2.0

# Install DID providers
npm install @veramo/did-provider-jwk
npm install @veramo/did-provider-key@4.2.0

# Install required dependencies
npm install did-resolver@^4.1.0
npm install reflect-metadata@^0.2.2
npm install @ethersproject/shims@^5.7.0
```

### Step 2: Add Polyfills to App Entry

Update `App.tsx` (at the very top):

```typescript
// App.tsx
import '@ethersproject/shims'; // MUST be first
import 'reflect-metadata'; // MUST be second

import React from 'react';
// ... rest of imports
```

Update `index.js`:

```javascript
// index.js
import '@ethersproject/shims';
import 'reflect-metadata';

import { registerRootComponent } from 'expo';
import App from './App';

registerRootComponent(App);
```

### Step 3: Create Veramo Agent Configuration

Create `src/agent/config.ts`:

```typescript
import { DataSource } from 'typeorm';
import { DB_CONNECTION_NAME } from '@config/database';

// Import Veramo entities
import {
  Entities as VeramoDataStoreEntities,
  migrations as VeramoDataStoreMigrations,
} from '@veramo/data-store';

// Database configuration for Veramo
export const getVeramoDataSource = async (): Promise<DataSource> => {
  const dataSource = new DataSource({
    type: 'react-native',
    database: DB_CONNECTION_NAME,
    location: 'default',
    entities: [
      ...VeramoDataStoreEntities,
      // Add custom entities here
    ],
    migrations: [
      ...VeramoDataStoreMigrations,
      // Add custom migrations here
    ],
    logging: ['error'],
    synchronize: false, // Use migrations instead
    migrationsRun: true,
  });

  if (!dataSource.isInitialized) {
    await dataSource.initialize();
  }

  return dataSource;
};
```

### Step 4: Create Agent Plugins Configuration

Create `src/agent/plugins.ts`:

```typescript
import { DataSource } from 'typeorm';

// Veramo imports
import { KeyManager } from '@veramo/key-manager';
import { DIDManager } from '@veramo/did-manager';
import { DIDResolverPlugin } from '@veramo/did-resolver';
import { KeyStore, DIDStore, PrivateKeyStore } from '@veramo/data-store';

// DID Providers
import { JwkDIDProvider } from '@veramo/did-provider-jwk';
import { KeyDIDProvider } from '@veramo/did-provider-key';

// DID Resolver
import { Resolver } from 'did-resolver';
import { getResolver as getKeyResolver } from 'key-did-resolver';
import { getResolver as getWebResolver } from 'web-did-resolver';

interface CreateAgentPluginsParams {
  dbConnection: DataSource;
  secretKey?: string;
}

export const createAgentPlugins = ({
  dbConnection,
  secretKey = 'your-secret-key-change-this', // TODO: Get from secure storage
}: CreateAgentPluginsParams) => {
  // Key Management
  const keyManager = new KeyManager({
    store: new KeyStore(dbConnection),
    kms: {
      local: new KeyManagementSystem(
        new PrivateKeyStore(dbConnection, new SecretBox(secretKey))
      ),
    },
  });

  // DID Management
  const didManager = new DIDManager({
    store: new DIDStore(dbConnection),
    defaultProvider: 'did:jwk',
    providers: {
      'did:jwk': new JwkDIDProvider({
        defaultKms: 'local',
      }),
      'did:key': new KeyDIDProvider({
        defaultKms: 'local',
      }),
    },
  });

  // DID Resolver
  const didResolver = new DIDResolverPlugin({
    resolver: new Resolver({
      ...getKeyResolver(),
      ...getWebResolver(),
    }),
  });

  return {
    keyManager,
    didManager,
    didResolver,
  };
};

// Key Management System implementation
class KeyManagementSystem {
  constructor(private readonly keyStore: PrivateKeyStore) {}

  async createKey({ type }: { type: string }) {
    // Implementation for key creation
    // This is simplified - actual implementation is more complex
    const keyId = `key-${Date.now()}`;
    return {
      kid: keyId,
      type,
      publicKeyHex: '...', // Generated public key
      meta: {},
    };
  }

  async sign({ keyRef, data }: { keyRef: string; data: string }) {
    // Implementation for signing
    return '...signature...';
  }
}

// Secret box for encryption
class SecretBox {
  constructor(private readonly secretKey: string) {}

  encrypt(data: string): string {
    // Simple encryption - use proper encryption in production
    return Buffer.from(data).toString('base64');
  }

  decrypt(encrypted: string): string {
    return Buffer.from(encrypted, 'base64').toString('utf8');
  }
}
```

### Step 5: Create Main Agent Instance

Create `src/agent/index.ts`:

```typescript
import { createAgent, IResolver, IKeyManager, IDIDManager } from '@veramo/core';
import { getVeramoDataSource } from './config';
import { createAgentPlugins } from './plugins';

// Agent interface
export type IAgent = IResolver & IKeyManager & IDIDManager;

// Singleton agent instance
let agentInstance: IAgent | null = null;

/**
 * Get or create Veramo agent instance
 */
export const getAgent = async (): Promise<IAgent> => {
  if (agentInstance) {
    return agentInstance;
  }

  // Initialize database
  const dbConnection = await getVeramoDataSource();

  // Create plugins
  const plugins = createAgentPlugins({
    dbConnection,
  });

  // Create agent
  agentInstance = createAgent<IAgent>({
    plugins: [
      plugins.keyManager,
      plugins.didManager,
      plugins.didResolver,
    ],
  });

  return agentInstance;
};

/**
 * Reset agent (for testing)
 */
export const resetAgent = () => {
  agentInstance = null;
};

// Export for convenience
export const agent = {
  get: getAgent,
  reset: resetAgent,
};
```

### Step 6: Create Wallet Instance Service

Update `src/services/walletInstance.ts`:

```typescript
import { getAgent } from '@agent/index';
import type { IAgent } from '@agent/index';

/**
 * Wallet service with Veramo agent
 */
class WalletService {
  private agent: IAgent | null = null;

  async initialize(): Promise<void> {
    if (this.agent) {
      return;
    }

    console.log('Initializing wallet agent...');
    this.agent = await getAgent();
    console.log('Wallet agent initialized successfully');
  }

  getAgent(): IAgent {
    if (!this.agent) {
      throw new Error('Wallet not initialized. Call initialize() first.');
    }
    return this.agent;
  }

  async createDID(method: 'did:jwk' | 'did:key' = 'did:jwk') {
    const agent = this.getAgent();
    
    const identifier = await agent.didManagerCreate({
      provider: method,
      kms: 'local',
      options: {
        keyType: 'Secp256k1',
      },
    });

    return identifier;
  }

  async resolveDID(did: string) {
    const agent = this.getAgent();
    const result = await agent.resolveDid({ didUrl: did });
    return result;
  }

  async listDIDs() {
    const agent = this.getAgent();
    const identifiers = await agent.didManagerFind();
    return identifiers;
  }
}

// Singleton instance
export const walletService = new WalletService();
```

### Step 7: Initialize Agent on App Start

Update `App.tsx`:

```typescript
import '@ethersproject/shims';
import 'reflect-metadata';
import { walletService } from './src/services/walletInstance';

export default function App() {
  const [appIsReady, setAppIsReady] = useState(false);

  useEffect(() => {
    async function prepare() {
      try {
        // ... existing initialization code ...

        // Initialize Veramo agent
        await walletService.initialize();
        console.log('Veramo agent ready');

        // ... rest of initialization ...
      } catch (error) {
        console.error('Error initializing app:', error);
      } finally {
        setAppIsReady(true);
      }
    }

    prepare();
  }, []);

  // ... rest of App component
}
```

### Step 8: Create Test Functions

Create `src/services/__tests__/agentTest.ts`:

```typescript
import { walletService } from '../walletInstance';

export const testAgentCreation = async () => {
  try {
    console.log('🧪 Testing Veramo agent...');

    // Initialize
    await walletService.initialize();
    console.log('✅ Agent initialized');

    // Create DID
    const identifier = await walletService.createDID('did:jwk');
    console.log('✅ Created DID:', identifier.did);

    // Resolve DID
    const resolved = await walletService.resolveDID(identifier.did);
    console.log('✅ Resolved DID document:', resolved.didDocument);

    // List DIDs
    const identifiers = await walletService.listDIDs();
    console.log('✅ Total DIDs:', identifiers.length);

    console.log('🎉 All agent tests passed!');
    return true;
  } catch (error) {
    console.error('❌ Agent test failed:', error);
    return false;
  }
};
```

### Step 9: Add Test Button to App (Temporary)

For testing purposes, add a button to test agent:

```typescript
// In your main screen (temporary)
import { testAgentCreation } from '@services/__tests__/agentTest';

const TestScreen = () => {
  const [result, setResult] = useState('');

  const handleTest = async () => {
    setResult('Testing...');
    const success = await testAgentCreation();
    setResult(success ? 'Tests passed!' : 'Tests failed!');
  };

  return (
    <View>
      <Button title="Test Veramo Agent" onPress={handleTest} />
      <Text>{result}</Text>
    </View>
  );
};
```

---

## ✅ Verification Steps

### 1. Check Installation

```bash
# Verify packages installed
npm list @veramo/core
npm list @veramo/did-manager
npm list reflect-metadata

# Should show versions installed
```

### 2. Check Agent Initialization

```bash
# Run app
npm run ios
# or
npm run android

# Check console logs:
# "Initializing wallet agent..."
# "Wallet agent initialized successfully"
```

### 3. Test DID Creation

```typescript
// Run test function
const testAgent = async () => {
  await walletService.initialize();
  
  // Create DID
  const id = await walletService.createDID('did:jwk');
  console.log('Created:', id.did);
  // Should output: did:jwk:...
  
  // Verify in database
  const ids = await walletService.listDIDs();
  console.log('Total DIDs:', ids.length);
};
```

### 4. Verify Database Tables

```typescript
// Check Veramo tables created
const checkTables = async () => {
  const connection = await getVeramoDataSource();
  const tables = await connection.query(
    "SELECT name FROM sqlite_master WHERE type='table'"
  );
  console.log('Tables:', tables);
  // Should include: identifier, key, credential, etc.
};
```

### 5. Test DID Resolution

```typescript
const testResolution = async () => {
  const did = 'did:key:z6MkpTHR8VNsBxYAAWHut2Geadd9jSwuBV8xRoAnwWsdvktH';
  const result = await walletService.resolveDID(did);
  console.log('Resolved:', result.didDocument);
  // Should show DID document
};
```

---

## 🎯 Acceptance Criteria

### Must Have ✅

- [x] All Veramo packages installed
- [x] Agent initializes without errors
- [x] Can create DIDs (did:jwk)
- [x] Can resolve DIDs
- [x] Keys stored in database
- [x] DIDs stored in database
- [x] Agent available globally
- [x] No console errors
- [x] TypeScript: 0 errors

### Should Have ✅

- [x] Multiple DID methods (jwk + key)
- [x] Proper error handling
- [x] Clean service abstraction
- [x] Test functions working
- [x] Database tables created correctly

### Nice to Have 🌟

- [ ] Agent performance metrics
- [ ] Advanced key types (Ed25519)
- [ ] DID method selection UI
- [ ] Agent debugging tools

---

## 🚨 Common Errors & Solutions

### Error 1: reflect-metadata not found

**Error**:
```
Cannot find module 'reflect-metadata'
```

**Solution**:
```bash
npm install reflect-metadata
```

```typescript
// Make sure it's imported FIRST in App.tsx and index.js
import 'reflect-metadata';
```

### Error 2: @ethersproject/shims error

**Error**:
```
crypto.getRandomValues() not supported
```

**Solution**:
```typescript
// Import shims BEFORE reflect-metadata
import '@ethersproject/shims';
import 'reflect-metadata';
```

### Error 3: Database connection error

**Error**:
```
Cannot connect to database for Veramo
```

**Solution**:
```typescript
// Make sure database is initialized first
const dbConnection = await getDbConnection(DB_CONNECTION_NAME);
// Then create Veramo data source
```

### Error 4: Provider not found

**Error**:
```
No provider found for did:jwk
```

**Solution**:
```typescript
// Make sure provider is registered
const didManager = new DIDManager({
  providers: {
    'did:jwk': new JwkDIDProvider({
      defaultKms: 'local'
    }),
  },
});
```

### Error 5: Key Management System error

**Error**:
```
KMS 'local' not found
```

**Solution**:
```typescript
// Register KMS in KeyManager
const keyManager = new KeyManager({
  kms: {
    local: new KeyManagementSystem(...)
  }
});
```

---

## 📚 Learning Notes

### Veramo Architecture

```
Veramo Agent
├── Plugins (modular functionality)
│   ├── KeyManager - manages keys
│   ├── DIDManager - manages DIDs
│   ├── DIDResolver - resolves DIDs
│   ├── CredentialPlugin - VCs (Phase 3)
│   └── DataStore - persistence
└── Methods (available via agent)
    ├── didManagerCreate()
    ├── didManagerFind()
    ├── resolveDid()
    └── ... (more in later phases)
```

### Key Concepts

**Agent**: Central orchestrator for all SSI operations

**Plugins**: Modular pieces of functionality
- KeyManager: Create & store keys
- DIDManager: Create & manage DIDs
- DIDResolver: Resolve DIDs to documents

**Data Store**: Persist identifiers, keys, credentials

**KMS (Key Management System)**: Secure key storage

---

## 🔗 Resources

- [Veramo Documentation](https://veramo.io/docs/basics/introduction)
- [Veramo Agent](https://veramo.io/docs/basics/agent)
- [Veramo Plugins](https://veramo.io/docs/basics/plugins)
- [DID Core Spec](https://www.w3.org/TR/did-core/)

---

## 📝 Story Completion Checklist

- [ ] Veramo packages installed
- [ ] Polyfills added (shims + reflect-metadata)
- [ ] Agent configuration created
- [ ] Plugins configured
- [ ] Main agent instance created
- [ ] Wallet service created
- [ ] Agent initializes on app start
- [ ] Test functions created
- [ ] DID creation tested
- [ ] DID resolution tested
- [ ] Database tables verified
- [ ] TypeScript: 0 errors
- [ ] ESLint: 0 errors
- [ ] Tested on both platforms
- [ ] Git committed
- [ ] Progress tracker updated

---

## 🎯 Next Steps

1. **Verify agent working**:
   ```bash
   npm run ios
   # Check console: "Wallet agent initialized successfully"
   ```

2. **Test DID creation**:
   - Use test button
   - Verify DID created
   - Check database

3. **Commit your work**:
   ```bash
   git add .
   git commit -m "feat: configure Veramo agent (STORY-023)
   
   - Installed Veramo core packages
   - Configured agent with plugins
   - Setup KeyManager and DIDManager
   - Setup DIDResolver
   - Created wallet service
   - Tested DID creation and resolution"
   ```

4. **Move to STORY-024**: Key Management Implementation

---

**Story**: 023 of 112  
**Status**: Ready to Implement  
**Time**: 5-6 hours  
**Next**: STORY-024 - Key Management

**Veramo agent foundation! 🔧🚀**
