# STORY-055: Contact Creation from Issuer

**Phase**: 4 - Credential Issuance Flow  
**Story**: 055 of 112  
**Estimated Time**: 3-4 hours  
**Difficulty**: ⭐⭐⭐ Moderate

---

## 📚 Pemahaman yang Harus Didapat

### Apa yang Akan Kamu Pelajari?

1. **Contact Auto-Creation**
   - Extracting issuer info dari metadata
   - Creating contact entity automatically
   - Linking credentials ke contacts
   - Avoiding duplicates

2. **Contact Data Model**
   - Contact fields (name, DID, logo, type)
   - Relationship dengan credentials
   - Contact metadata storage

---

### 🎭 Analogi Dunia Nyata

**Bayangkan kamu terima ijazah dari universitas:**

```
Scenario 1: Tanpa Save Contact (Ribet):
───────────────────────────────────────
📜 Terima ijazah dari "Universitas XYZ"
⏰ 6 bulan kemudian...
👤 Employer: "Kami mau verify ijazah kamu"
😰 Kamu: "Eh... universitas mana ya? Lupa URLnya"
🔍 Google: "Universitas XYZ credential verification"
😓 "Websitenya apa ya? Ada banyak hasil..."
⏰ Waktu terbuang 30 menit

Scenario 2: Auto-Save Contact (Mudah):
──────────────────────────────────────
📜 Terima ijazah dari "Universitas XYZ"
✓ Wallet auto-save:
  - Nama: Universitas XYZ
  - Logo: 🏛️
  - DID: did:web:univ-xyz.edu
  - URL: https://univ-xyz.edu
  - Type: Issuer

⏰ 6 bulan kemudian...
👤 Employer: "Verify ijazah kamu"
✓ Buka wallet → Contacts → "Universitas XYZ"
✓ Klik → Lihat semua credentials dari mereka
✓ Share verification URL
⏰ Waktu: 10 detik! ⚡
```

**Sama seperti di HP:**

```
Terima Telpon Pertama Kali:
───────────────────────────
📞 +62812-3456-7890 calling...
✅ Angkat → "Halo, ini Bu Siti dari Bank XYZ"
❓ "Siapa ya nomornya?"

Save Contact:
─────────────
✓ Save: "Bu Siti - Bank XYZ"
✓ Foto: 🏦
✓ Jabatan: Customer Service

Telpon Berikutnya:
──────────────────
📞 "Bu Siti - Bank XYZ" calling...
✅ Langsung tahu siapa!
✅ Bisa lihat history komunikasi
✅ Trusted contact
```

**Sama seperti Contact Creation di Wallet:**

```
First Time Receive Credential:
───────────────────────────────
🏛️ Issuer: Universitas XYZ
❓ "Siapa issuer ini?"

Wallet Auto-Create Contact:
────────────────────────────
Extract dari Metadata:
✓ Name: "Universitas XYZ"
✓ Logo: https://univ-xyz.edu/logo.png
✓ DID: did:web:univ-xyz.edu  
✓ Type: Issuer
✓ Background Color: #003366
✓ Supported Credentials: [Degree, Transcript]

Save to Database:
─────────────────
contacts {
  id: "uuid-123",
  name: "Universitas XYZ",
  did: "did:web:univ-xyz.edu",
  logo: "...",
  type: "issuer"
}

Link Credential:
────────────────
credential.issuerId = "uuid-123"

Next Time:
──────────
✓ View all credentials from "Universitas XYZ"
✓ See issuer info instantly
✓ Trust relationship established
✓ Easy to contact for verification
```

**Benefits:**

```
Tanpa Contact Auto-Creation:
────────────────────────────
📜 Credential 1 dari: did:web:univ-xyz.edu
📜 Credential 2 dari: did:web:univ-xyz.edu
❓ "Ini DID siapa ya?"
❓ "Credentials dari issuer yang sama?"
😰 Susah track & organize

Dengan Contact Auto-Creation:
──────────────────────────────
🏛️ Universitas XYZ (2 credentials)
  📜 Bachelor Degree
  📜 Transcript
✓ Jelas issuer-nya siapa
✓ Easy to group credentials
✓ Professional display with logo
✓ Can view all credentials per issuer
```

**Contact Auto-Creation = Auto-Save Phonebook Entry**

- Pertama kali terima credential → Auto-save issuer info
- Next time → Langsung kenal siapa issuer-nya
- Bisa lihat semua credentials dari issuer yang sama
- Professional display dengan logo & branding

**Keuntungan:**
- ⚡ Save time - No manual entry
- 🎨 Better UX - Logo & colors
- 📊 Organization - Group by issuer
- 🔍 Easy verification - Contact info ready
- 📱 Professional - Like real contact app

---

## 🎯 Story Objectives

### Primary Goal
Auto-create contact entry untuk issuer saat credential issuance

### Specific Objectives
1. ✅ Check if issuer sudah ada di contacts
2. ✅ Extract issuer info dari metadata
3. ✅ Create contact entity
4. ✅ Store issuer DID
5. ✅ Link credentials ke contact
6. ✅ Update UI untuk show issuer info

---

## 📝 Implementation Steps

### Step 1: Update Contact Service

**Update**: `src/services/contactService.ts`

```typescript
/**
 * Contact Service
 * 
 * Manages issuer and verifier contacts
 */

import { AppDataSource } from '../database'
import { Contact } from '../entities/Contact'
import { IssuerMetadata, ParsedMetadata } from '../types/issuerMetadata'

class ContactService {
  private repository = AppDataSource.getRepository(Contact)

  /**
   * Find contact by DID
   */
  async findByDID(did: string): Promise<Contact | null> {
    return this.repository.findOne({
      where: { did }
    })
  }

  /**
   * Create contact dari issuer metadata
   */
  async createFromIssuer(metadata: ParsedMetadata): Promise<Contact> {
    try {
      console.log('[ContactService] Creating contact for:', metadata.issuerUrl)

      // Check if already exists
      const existing = await this.findByDID(metadata.issuerUrl)
      if (existing) {
        console.log('[ContactService] Contact already exists')
        return existing
      }

      // Create new contact
      const contact = this.repository.create({
        name: metadata.displayInfo.name,
        did: metadata.issuerUrl,
        logoUri: metadata.displayInfo.logo,
        type: 'issuer',
        metadata: {
          backgroundColor: metadata.displayInfo.backgroundColor,
          textColor: metadata.displayInfo.textColor,
          supportedCredentials: metadata.supportedCredentials
        }
      })

      await this.repository.save(contact)

      console.log('[ContactService] Contact created:', contact.id)

      return contact

    } catch (error) {
      console.error('[ContactService] Creation failed:', error)
      throw error
    }
  }

  /**
   * Get or create contact
   */
  async getOrCreateFromIssuer(metadata: ParsedMetadata): Promise<Contact> {
    const existing = await this.findByDID(metadata.issuerUrl)
    
    if (existing) {
      return existing
    }

    return this.createFromIssuer(metadata)
  }

  /**
   * Update contact activity timestamp
   */
  async updateActivity(contactId: string): Promise<void> {
    await this.repository.update(contactId, {
      lastInteraction: new Date()
    })
  }

  /**
   * Get all issuer contacts
   */
  async getIssuers(): Promise<Contact[]> {
    return this.repository.find({
      where: { type: 'issuer' },
      order: { lastInteraction: 'DESC' }
    })
  }
}

export const contactService = new ContactService()
```

---

### Step 2: Update Contact Entity

**Update**: `src/entities/Contact.ts`

```typescript
/**
 * Contact Entity (Issuer/Verifier)
 */

import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn } from 'typeorm'

@Entity('contacts')
export class Contact {
  @PrimaryGeneratedColumn('uuid')
  id: string

  @Column()
  name: string

  @Column({ unique: true })
  did: string

  @Column({ nullable: true })
  logoUri?: string

  @Column()
  type: 'issuer' | 'verifier' | 'both'

  @Column('json', { nullable: true })
  metadata?: {
    backgroundColor?: string
    textColor?: string
    supportedCredentials?: string[]
    [key: string]: any
  }

  @Column({ nullable: true })
  lastInteraction?: Date

  @CreateDateColumn()
  createdAt: Date

  @UpdateDateColumn()
  updatedAt: Date
}
```

---

### Step 3: Integrate dengan Issuance Flow

**Update**: `src/services/oid4vci/OID4VCIService.ts`

```typescript
import { contactService } from '../contactService'

class OID4VCIService {
  // ... existing code ...

  /**
   * Get or create issuer contact
   */
  async ensureIssuerContact(metadata: ParsedMetadata): Promise<Contact> {
    return contactService.getOrCreateFromIssuer(metadata)
  }

  /**
   * Complete issuance flow dengan contact creation
   */
  async issueCredential(offerUri: string, userPin?: string) {
    try {
      // 1. Parse offer
      const offer = await this.parseOffer(offerUri)
      
      // 2. Resolve metadata
      const metadata = await this.resolveMetadata(offer.issuerUrl)
      
      // 3. Create/get contact
      const contact = await this.ensureIssuerContact(metadata)
      console.log('[OID4VCI] Issuer contact:', contact.name)
      
      // 4. Get token
      const token = await this.requestToken(offer, metadata, userPin)
      
      // 5. Generate proof
      const did = await this.getDefaultDID()
      const proof = await this.generateProof(
        did,
        metadata.issuerUrl,
        token.c_nonce
      )
      
      // 6. Request credential
      const credential = await this.requestCredential(
        metadata,
        token,
        proof,
        offer.credentialTypes[0]
      )
      
      // 7. Store credential (will link to contact)
      await this.storeCredential(credential, contact.id)
      
      // 8. Update contact activity
      await contactService.updateActivity(contact.id)
      
      return {
        credential,
        contact,
        success: true
      }

    } catch (error) {
      console.error('[OID4VCI] Issuance failed:', error)
      throw error
    }
  }

  private async getDefaultDID(): Promise<string> {
    // Get from identity service
    const identities = await agent.didManagerFind()
    return identities[0]?.did
  }

  private async storeCredential(credential: any, contactId: string): Promise<void> {
    // Store using credential service with contact link
    // Implementation in credential service
  }
}
```

---

### Step 4: Update Credential Entity

**Update**: `src/entities/Credential.ts`

Add contact relationship:

```typescript
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne, JoinColumn } from 'typeorm'
import { Contact } from './Contact'

@Entity('credentials')
export class Credential {
  // ... existing fields ...

  @Column({ nullable: true })
  issuerId?: string

  @ManyToOne(() => Contact, { nullable: true })
  @JoinColumn({ name: 'issuerId' })
  issuer?: Contact

  // ... rest of fields ...
}
```

---

### Step 5: Display Issuer in Credential Details

**Update**: `src/screens/CredentialDetailsScreen.tsx`

```typescript
import { contactService } from '../services/contactService'

export const CredentialDetailsScreen: React.FC = ({ route }) => {
  const { credentialId } = route.params
  const [credential, setCredential] = useState(null)
  const [issuer, setIssuer] = useState(null)

  useEffect(() => {
    loadCredential()
  }, [credentialId])

  const loadCredential = async () => {
    const cred = await credentialService.getById(credentialId)
    setCredential(cred)

    if (cred.issuerId) {
      const issuerContact = await contactService.findById(cred.issuerId)
      setIssuer(issuerContact)
    }
  }

  return (
    <View>
      {/* Credential info */}
      
      {/* Issuer info */}
      {issuer && (
        <View style={styles.issuerSection}>
          <Text style={styles.label}>Issued By:</Text>
          
          {issuer.logoUri && (
            <Image source={{ uri: issuer.logoUri }} style={styles.issuerLogo} />
          )}
          
          <Text style={styles.issuerName}>{issuer.name}</Text>
          <Text style={styles.issuerDid}>{issuer.did}</Text>
          
          <TouchableOpacity onPress={() => navigateToIssuer(issuer.id)}>
            <Text style={styles.link}>View Issuer Details →</Text>
          </TouchableOpacity>
        </View>
      )}
    </View>
  )
}
```

---

## ✅ Verification Steps

### Test 1: Create Contact from Metadata

```typescript
const contact = await contactService.createFromIssuer(parsedMetadata)

expect(contact.name).toBe(parsedMetadata.displayInfo.name)
expect(contact.did).toBe(parsedMetadata.issuerUrl)
expect(contact.type).toBe('issuer')
```

### Test 2: Avoid Duplicates

```typescript
const contact1 = await contactService.getOrCreateFromIssuer(metadata)
const contact2 = await contactService.getOrCreateFromIssuer(metadata)

expect(contact1.id).toBe(contact2.id)
```

### Test 3: Credential Linked to Contact

```typescript
const credential = await credentialService.getById(credId)
expect(credential.issuerId).toBeDefined()

const issuer = await contactService.findById(credential.issuerId)
expect(issuer.type).toBe('issuer')
```

---

## 🎯 Acceptance Criteria

- [x] Contact auto-created saat issuance
- [x] Issuer info extracted dari metadata
- [x] No duplicate contacts
- [x] Credential linked ke issuer contact
- [x] Contact displayed di credential details
- [x] Logo dan branding preserved
- [x] Activity timestamp updated

---

## 🎓 Key Learnings

- Auto-creation patterns
- Entity relationships (Credential → Contact)
- Data extraction dari metadata
- Duplicate prevention
- Activity tracking

---

**Story Status**: 📝 READY TO IMPLEMENT  
**Next**: STORY-056 - Credential Acceptance Flow
