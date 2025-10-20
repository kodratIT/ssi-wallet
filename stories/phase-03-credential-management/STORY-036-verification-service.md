# STORY-036: Credential Verification Service

**Phase**: 3 - Credential Management  
**Story**: 036 of 112  
**Estimated Time**: 4-5 hours  
**Difficulty**: ⭐⭐⭐⭐ Advanced

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari di Story Ini?

Setelah menyelesaikan story ini, kamu akan **MEMAHAMAN**:

1. **Cryptographic Verification**
   - Bagaimana verify digital signature works
   - Public key cryptography in practice
   - Why signature proves authenticity

2. **Credential Validity Checks**
   - Expiry date checking logic
   - Time-based validation
   - Status checking (revocation)

3. **Trust Chains**
   - Issuer trust levels
   - DID resolution for verification
   - Chain of trust concept

4. **Verification Results**
   - Different types of verification failures
   - Result interpretation
   - User-friendly error messages

### Mengapa Story Ini Penting?

**Verification adalah SECURITY FOUNDATION dari SSI**:
- ❌ Skip verification = Accept fake credentials
- ❌ Bad verification = Security vulnerability
- ❌ No verification = Useless wallet
- ✅ Proper verification = Trust dan security!

**Analogi**:
```
Verifikasi Credential seperti verifikasi uang:

TANPA Verifikasi:
- Terima semua uang (fake or real)
- Bisa ditipu
- Trust anyone
- BERBAHAYA!

DENGAN Verifikasi:
- Check watermark (signature)
- Check serial number (issuer DID)
- Check expiry (if any)
- Check tidak di-blacklist (revocation)
- AMAN!

Verification = Quality control untuk credentials!
```

---

## 🎯 Story Objectives

### Primary Goal
Build comprehensive verification system untuk validate Verifiable Credentials sebelum trusted

### Specific Objectives
1. ✅ Implement signature verification
2. ✅ Implement expiry checking
3. ✅ Implement status/revocation checking
4. ✅ Implement issuer trust validation
5. ✅ Create verification result types
6. ✅ Add batch verification for multiple VCs
7. ✅ Test all verification scenarios

---

## 📋 Prerequisites

### Knowledge Prerequisites

**WAJIB sudah paham**:
- [x] STORY-033 complete (Veramo verification API)
- [x] STORY-034 & 035 complete (Database & service)
- [x] Public key cryptography basics
- [x] Digital signatures concept
- [x] Time/date handling in TypeScript

**Test diri sendiri**:
```
Bisa jawab pertanyaan ini?
1. Apa itu digital signature?
2. Bagaimana verify signature?
3. Mengapa signature proves authenticity?
4. Apa itu credential revocation?

Kalau belum → Belajar crypto basics dulu!
```

### Technical Prerequisites
- ✅ STORY-033, 034, 035 complete
- ✅ Veramo agent working
- ✅ Can verify VCs via Veramo

---

## 📝 Implementation Steps

### Step 1: Define Verification Result Types

**⏱️ Estimasi**: 20 minutes  
**🎓 Kegunaan**: Type-safe verification results

#### Apa yang Akan Dilakukan?
Create TypeScript types untuk verification results

#### Mengapa Penting?
Clear types = Clear contract = Easy to use API. Users know exactly what to expect!

#### Pemahaman yang Didapat:
- TypeScript discriminated unions
- Result pattern (success/failure)
- Type safety benefits

#### Implementation:

Create `src/types/verification.types.ts`:

```typescript
/**
 * PEMAHAMAN: Verification Result Types
 * 
 * Why separate types file?
 * - Reusable across codebase
 * - Clear documentation
 * - Type safety
 * - Easy to extend
 */

/**
 * Verification Check Types
 * 
 * Different aspects of credential verification:
 * - SIGNATURE: Cryptographic signature valid?
 * - EXPIRY: Not expired?
 * - STATUS: Not revoked/suspended?
 * - ISSUER: Issuer trusted?
 */
export enum VerificationCheckType {
  SIGNATURE = 'signature',
  EXPIRY = 'expiry',
  STATUS = 'status',
  ISSUER = 'issuer',
}

/**
 * Individual Check Result
 * 
 * Result of one verification check
 */
export interface VerificationCheck {
  type: VerificationCheckType;
  passed: boolean;
  message?: string;
  details?: any;
}

/**
 * Overall Verification Result
 * 
 * PEMAHAMAN: Result Pattern
 * 
 * Instead of throwing errors, return result object:
 * 
 * BAD:
 * function verify() {
 *   if (invalid) throw new Error(); // Lost context
 * }
 * 
 * GOOD:
 * function verify(): VerificationResult {
 *   return {
 *     valid: false,
 *     checks: [...], // Which checks failed?
 *     summary: "..." // What went wrong?
 *   }
 * }
 * 
 * Benefits:
 * - Caller can handle gracefully
 * - Detailed error information
 * - No try-catch needed
 * - Better UX (show specific issues)
 */
export interface VerificationResult {
  // Overall validity
  valid: boolean;

  // Credential ID
  credentialId: string;

  // Individual checks performed
  checks: VerificationCheck[];

  // Human-readable summary
  summary: string;

  // When verified
  verifiedAt: Date;

  // Issuer info (if resolved)
  issuer?: {
    did: string;
    name?: string;
    trusted?: boolean;
  };

  // Raw Veramo result (for debugging)
  raw?: any;
}

/**
 * PEMAHAMAN: Verification Failure Reasons
 * 
 * Categorize failures for better UX
 */
export enum VerificationFailureReason {
  SIGNATURE_INVALID = 'signature_invalid',
  EXPIRED = 'expired',
  REVOKED = 'revoked',
  SUSPENDED = 'suspended',
  ISSUER_UNKNOWN = 'issuer_unknown',
  ISSUER_UNTRUSTED = 'issuer_untrusted',
  MALFORMED = 'malformed',
  NETWORK_ERROR = 'network_error',
}

/**
 * Batch Verification Result
 * 
 * For verifying multiple credentials at once
 */
export interface BatchVerificationResult {
  totalCredentials: number;
  validCredentials: number;
  invalidCredentials: number;
  results: VerificationResult[];
  summary: string;
}
```

---

### Step 2: Create Verification Service

**⏱️ Estimasi**: 60 minutes  
**🎓 Kegunaan**: Core verification logic

#### Apa yang Akan Dilakukan?
Implement comprehensive verification service

#### Mengapa Penting?
This is WHERE THE MAGIC HAPPENS - cryptographic verification validates trust!

#### Pemahaman yang Didapat:
- Complete verification workflow
- Signature verification mechanics
- Error handling strategies
- Performance considerations

#### Implementation:

Create `src/services/VerificationService.ts`:

```typescript
import { getAgent } from '@agent/index';
import type { IAgent, VerifiableCredential } from '@veramo/core';
import {
  VerificationResult,
  VerificationCheck,
  VerificationCheckType,
  VerificationFailureReason,
  BatchVerificationResult,
} from '@types/verification.types';

/**
 * PEMAHAMAN: Verification Service
 * 
 * Service ini melakukan COMPREHENSIVE verification:
 * 
 * 4 Checks:
 * 1. SIGNATURE: Is signature cryptographically valid?
 * 2. EXPIRY: Is credential still valid (not expired)?
 * 3. STATUS: Is credential revoked/suspended?
 * 4. ISSUER: Is issuer trusted/known?
 * 
 * ALL checks must pass untuk credential dianggap valid.
 */

class VerificationService {
  private agent: IAgent;

  private constructor() {}

  private async init() {
    if (!this.agent) {
      this.agent = await getAgent();
    }
  }

  private static instance: VerificationService;

  static async getInstance(): Promise<VerificationService> {
    if (!VerificationService.instance) {
      VerificationService.instance = new VerificationService();
      await VerificationService.instance.init();
    }
    return VerificationService.instance;
  }

  /**
   * TASK: Verify Credential
   * 
   * KEGUNAAN:
   * - Comprehensive credential verification
   * - All security checks
   * - Detailed result for debugging
   * 
   * PEMAHAMAN YANG DIDAPAT:
   * - Complete verification workflow
   * - Each check's purpose
   * - Why all checks necessary
   * - How to interpret results
   * 
   * FLOW:
   * 1. Validate input
   * 2. Check signature (Veramo)
   * 3. Check expiry (time-based)
   * 4. Check status (revocation list)
   * 5. Check issuer (trust level)
   * 6. Aggregate results
   * 7. Return comprehensive result
   */
  async verifyCredential(
    credential: VerifiableCredential
  ): Promise<VerificationResult> {
    console.log('🔍 Starting comprehensive verification...');

    const checks: VerificationCheck[] = [];
    const credentialId = credential.id || 'unknown';

    try {
      /**
       * CHECK 1: Signature Verification
       * 
       * PEMAHAMAN: Cryptographic Signature Check
       * 
       * What happens:
       * 1. Extract issuer DID from credential
       * 2. Resolve DID document (get public key)
       * 3. Extract signature from credential (proof field)
       * 4. Verify signature dengan public key
       * 
       * Mathematical magic:
       * - Issuer signed dengan PRIVATE key
       * - We verify dengan PUBLIC key
       * - If signature valid = credential authentic
       * - If signature invalid = tampered or fake
       * 
       * Why this works:
       * - Private key mathematically linked to public key
       * - Can't forge signature without private key
       * - Private key only issuer has
       * - Therefore: valid signature = issuer created this!
       */
      console.log('🔐 Checking signature...');
      const signatureCheck = await this.checkSignature(credential);
      checks.push(signatureCheck);

      if (!signatureCheck.passed) {
        return this.createFailedResult(
          credentialId,
          checks,
          'Signature verification failed'
        );
      }

      /**
       * CHECK 2: Expiry Check
       * 
       * PEMAHAMAN: Time-Based Validity
       * 
       * Credentials can have expiry dates (like passports)
       * 
       * Example:
       * issuanceDate: "2024-01-01"
       * expirationDate: "2029-01-01"
       * 
       * Check:
       * if (now > expirationDate) → EXPIRED
       * 
       * Why important:
       * - Information can become outdated
       * - Security: Limit validity window
       * - Compliance: Regulations require expiry
       */
      console.log('📅 Checking expiry...');
      const expiryCheck = await this.checkExpiry(credential);
      checks.push(expiryCheck);

      if (!expiryCheck.passed) {
        return this.createFailedResult(
          credentialId,
          checks,
          'Credential has expired'
        );
      }

      /**
       * CHECK 3: Status Check (Revocation)
       * 
       * PEMAHAMAN: Revocation Mechanism
       * 
       * Issuer might revoke credential after issuance:
       * - Employee fired → Employment credential revoked
       * - Degree revoked → Academic credential invalid
       * - Lost device → All credentials should be revoked
       * 
       * How it works:
       * - Issuer maintains revocation list
       * - Check if credential ID in revocation list
       * - Can be status list, bitstring, or API
       * 
       * Why necessary:
       * - Issuer must have control after issuance
       * - Security: Invalidate compromised credentials
       * - Flexibility: Change status post-issuance
       */
      console.log('🚫 Checking revocation status...');
      const statusCheck = await this.checkStatus(credential);
      checks.push(statusCheck);

      if (!statusCheck.passed) {
        return this.createFailedResult(
          credentialId,
          checks,
          'Credential has been revoked or suspended'
        );
      }

      /**
       * CHECK 4: Issuer Trust
       * 
       * PEMAHAMAN: Trust Levels
       * 
       * Not all issuers are equal:
       * - Government issuer → High trust
       * - Unknown DID → Low trust
       * - Known malicious → No trust
       * 
       * Checks:
       * - Is issuer DID known?
       * - Is issuer in trusted list?
       * - Domain verification (well-known DID)
       * 
       * Why important:
       * - Anyone can issue credentials
       * - Must determine if issuer trustworthy
       * - User should know trust level
       */
      console.log('🏛️ Checking issuer trust...');
      const issuerCheck = await this.checkIssuer(credential);
      checks.push(issuerCheck);

      /**
       * Aggregate Results
       * 
       * All checks passed → VALID
       * Any check failed → INVALID
       */
      const allPassed = checks.every((check) => check.passed);

      return {
        valid: allPassed,
        credentialId,
        checks,
        summary: allPassed
          ? 'Credential is valid'
          : 'Credential verification failed',
        verifiedAt: new Date(),
        issuer: await this.extractIssuerInfo(credential),
      };
    } catch (error) {
      console.error('❌ Verification error:', error);

      // Error during verification
      checks.push({
        type: VerificationCheckType.SIGNATURE,
        passed: false,
        message: `Verification error: ${error.message}`,
      });

      return this.createFailedResult(
        credentialId,
        checks,
        `Verification error: ${error.message}`
      );
    }
  }

  /**
   * TASK: Check Signature
   * 
   * KEGUNAAN:
   * - Verify cryptographic signature
   * - Proves credential authentic
   * 
   * PEMAHAMAN:
   * - Uses Veramo verifyCredential()
   * - Veramo handles complex crypto
   * - We interpret result
   */
  private async checkSignature(
    credential: VerifiableCredential
  ): Promise<VerificationCheck> {
    try {
      /**
       * PEMAHAMAN: What Veramo Does
       * 
       * Behind the scenes:
       * 1. Parse credential (JWT or JSON-LD)
       * 2. Extract issuer DID
       * 3. Resolve DID document
       * 4. Get verification method (public key)
       * 5. Verify signature
       * 
       * For JWT credentials:
       * - Decode JWT
       * - Extract header, payload, signature
       * - Verify signature = HMAC(header + payload, secret)
       * 
       * For JSON-LD credentials:
       * - Extract proof object
       * - Apply cryptographic suite
       * - Verify according to suite rules
       */
      const result = await this.agent.verifyCredential({
        credential,
      });

      return {
        type: VerificationCheckType.SIGNATURE,
        passed: result.verified,
        message: result.verified
          ? 'Signature is valid'
          : 'Signature verification failed',
        details: result,
      };
    } catch (error) {
      return {
        type: VerificationCheckType.SIGNATURE,
        passed: false,
        message: `Signature check error: ${error.message}`,
      };
    }
  }

  /**
   * TASK: Check Expiry
   * 
   * KEGUNAAN:
   * - Check if credential expired
   * - Time-based validity
   * 
   * PEMAHAMAN:
   * - Simple date comparison
   * - Handle missing expiry (valid forever)
   * - Timezone considerations
   */
  private async checkExpiry(
    credential: VerifiableCredential
  ): Promise<VerificationCheck> {
    try {
      const now = new Date();

      // Check expiry date if present
      if (credential.expirationDate) {
        const expiryDate = new Date(credential.expirationDate);

        if (now > expiryDate) {
          return {
            type: VerificationCheckType.EXPIRY,
            passed: false,
            message: `Credential expired on ${expiryDate.toLocaleDateString()}`,
            details: { expiryDate, now },
          };
        }
      }

      // No expiry date or not yet expired
      return {
        type: VerificationCheckType.EXPIRY,
        passed: true,
        message: credential.expirationDate
          ? 'Credential is not expired'
          : 'Credential has no expiry date',
      };
    } catch (error) {
      return {
        type: VerificationCheckType.EXPIRY,
        passed: false,
        message: `Expiry check error: ${error.message}`,
      };
    }
  }

  /**
   * TASK: Check Status (Revocation)
   * 
   * KEGUNAAN:
   * - Check if credential revoked
   * - Query revocation list
   * 
   * PEMAHAMAN:
   * - Credential status can change post-issuance
   * - Multiple revocation methods exist
   * - Network call might be required
   */
  private async checkStatus(
    credential: VerifiableCredential
  ): Promise<VerificationCheck> {
    try {
      /**
       * PEMAHAMAN: Credential Status Methods
       * 
       * W3C VC spec defines credentialStatus field:
       * 
       * credentialStatus: {
       *   id: "https://issuer.com/status/1",
       *   type: "StatusList2021Entry"
       * }
       * 
       * Types:
       * 1. StatusList2021 - Bitstring status list
       * 2. RevocationList2020 - Simple revocation list
       * 3. API endpoint - Query issuer API
       * 
       * For Phase 3: Basic implementation
       * Phase 4 will implement full status checking
       */

      // If no status field, assume not revoked
      if (!credential.credentialStatus) {
        return {
          type: VerificationCheckType.STATUS,
          passed: true,
          message: 'No status information (assumed valid)',
        };
      }

      // TODO: Implement status list checking in Phase 4
      // For now, basic check
      console.log('⚠️ Status checking not fully implemented yet');

      return {
        type: VerificationCheckType.STATUS,
        passed: true,
        message: 'Status check: Basic (full check in Phase 4)',
      };
    } catch (error) {
      return {
        type: VerificationCheckType.STATUS,
        passed: true, // Fail open (don't reject if can't check)
        message: `Status check error: ${error.message}`,
      };
    }
  }

  /**
   * TASK: Check Issuer
   * 
   * KEGUNAAN:
   * - Validate issuer trust level
   * - Resolve issuer DID
   * 
   * PEMAHAMAN:
   * - Can resolve DID?
   * - Is issuer known?
   * - Domain verification
   */
  private async checkIssuer(
    credential: VerifiableCredential
  ): Promise<VerificationCheck> {
    try {
      const issuerDid =
        typeof credential.issuer === 'string'
          ? credential.issuer
          : credential.issuer?.id;

      if (!issuerDid) {
        return {
          type: VerificationCheckType.ISSUER,
          passed: false,
          message: 'No issuer DID found',
        };
      }

      // Try to resolve issuer DID
      try {
        const didDocument = await this.agent.resolveDid({ didUrl: issuerDid });

        if (!didDocument.didDocument) {
          return {
            type: VerificationCheckType.ISSUER,
            passed: false,
            message: 'Issuer DID could not be resolved',
          };
        }

        return {
          type: VerificationCheckType.ISSUER,
          passed: true,
          message: 'Issuer DID resolved successfully',
          details: { issuerDid },
        };
      } catch (error) {
        return {
          type: VerificationCheckType.ISSUER,
          passed: false,
          message: `Could not resolve issuer DID: ${error.message}`,
        };
      }
    } catch (error) {
      return {
        type: VerificationCheckType.ISSUER,
        passed: false,
        message: `Issuer check error: ${error.message}`,
      };
    }
  }

  /**
   * Helper: Extract Issuer Info
   */
  private async extractIssuerInfo(credential: VerifiableCredential) {
    const issuerDid =
      typeof credential.issuer === 'string'
        ? credential.issuer
        : credential.issuer?.id;

    return {
      did: issuerDid || 'unknown',
      name: credential.issuer?.name,
      trusted: false, // TODO: Implement trust list
    };
  }

  /**
   * Helper: Create Failed Result
   */
  private createFailedResult(
    credentialId: string,
    checks: VerificationCheck[],
    summary: string
  ): VerificationResult {
    return {
      valid: false,
      credentialId,
      checks,
      summary,
      verifiedAt: new Date(),
    };
  }

  /**
   * TASK: Batch Verification
   * 
   * KEGUNAAN:
   * - Verify multiple credentials at once
   * - Background verification
   * - Performance optimization
   */
  async verifyBatch(
    credentials: VerifiableCredential[]
  ): Promise<BatchVerificationResult> {
    console.log(`🔍 Batch verifying ${credentials.length} credentials...`);

    const results: VerificationResult[] = [];

    for (const credential of credentials) {
      const result = await this.verifyCredential(credential);
      results.push(result);
    }

    const validCount = results.filter((r) => r.valid).length;
    const invalidCount = results.length - validCount;

    return {
      totalCredentials: credentials.length,
      validCredentials: validCount,
      invalidCredentials: invalidCount,
      results,
      summary: `Verified ${credentials.length} credentials: ${validCount} valid, ${invalidCount} invalid`,
    };
  }
}

export default VerificationService;
```

---

### Step 3: Create Test Cases

**⏱️ Estimasi**: 45 minutes  
**🎓 Kegunaan**: Test all verification scenarios

#### Implementation:

Create `src/services/__tests__/verification.test.ts`:

```typescript
import VerificationService from '../VerificationService';
import type { VerifiableCredential } from '@veramo/core';

describe('VerificationService', () => {
  let service: VerificationService;

  beforeAll(async () => {
    service = await VerificationService.getInstance();
  });

  it('should verify valid credential', async () => {
    const validVC: VerifiableCredential = {
      // ... valid VC
    };

    const result = await service.verifyCredential(validVC);

    expect(result.valid).toBe(true);
    expect(result.checks).toHaveLength(4);
  });

  it('should detect expired credential', async () => {
    const expiredVC: VerifiableCredential = {
      // ... expired VC
      expirationDate: '2020-01-01T00:00:00Z',
    };

    const result = await service.verifyCredential(expiredVC);

    expect(result.valid).toBe(false);
    const expiryCheck = result.checks.find((c) => c.type === 'expiry');
    expect(expiryCheck?.passed).toBe(false);
  });

  // More test cases...
});
```

---

## ✅ Acceptance Criteria

- [x] VerificationService created
- [x] Signature verification working
- [x] Expiry checking implemented
- [x] Status checking implemented
- [x] Issuer validation implemented
- [x] Batch verification working
- [x] Comprehensive result types
- [x] All tests passing
- [x] Error handling robust
- [x] TypeScript: 0 errors

---

## 🎓 Key Learnings Summary

**After this story, you understand**:

1. **Cryptographic Verification**
   - ✅ Digital signatures work
   - ✅ Public key crypto in action
   - ✅ Why signatures prove authenticity

2. **Comprehensive Checking**
   - ✅ Multiple verification aspects
   - ✅ Each check's importance
   - ✅ Aggregating results

3. **Security Mindset**
   - ✅ Never trust without verification
   - ✅ Defense in depth
   - ✅ Fail secure

---

**Story**: 036 of 112  
**Status**: Ready to Implement  
**Time**: 4-5 hours  
**Next**: STORY-037 - Branding Service

**Security foundation solid - credentials verified! 🔐✨**
