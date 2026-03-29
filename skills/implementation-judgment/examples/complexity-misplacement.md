# Complexity Misplacement — Scattered Validation and the Illusion of Defense in Depth

This contrast pair illustrates a common failure mode where partial
responsibility spread across multiple layers creates a false sense of safety
while making the system harder to understand, test, and maintain.

---

## Scenario

**Request:** "Add file upload validation for user avatars. Max 5MB, only
JPEG/PNG allowed, and the image must be at least 100x100 pixels."

**Current code shape:**
```typescript
// routes/profile.ts
router.post('/profile/avatar', upload.single('avatar'), async (req, res) => {
  const result = await ProfileService.updateAvatar(req.auth.userId, req.file)
  res.json(result)
})

// services/ProfileService.ts
class ProfileService {
  static async updateAvatar(userId: string, file: UploadedFile) {
    const url = await StorageService.upload(file, `avatars/${userId}`)
    await db.users.update(userId, { avatarUrl: url })
    return { avatarUrl: url }
  }
}

// services/StorageService.ts
class StorageService {
  static async upload(file: UploadedFile, path: string) {
    await s3.putObject({ Bucket: BUCKET, Key: path, Body: file.buffer })
    return `https://cdn.example.com/${path}`
  }
}
```

No validation exists yet. Three layers of code touch the file before it
reaches storage.

---

## Bad Case — Every Layer Does a Little Validation

**Reasoning:** "Defense in depth! Each layer should validate what it can."

**Implementation:**
```typescript
// routes/profile.ts — checks size at the HTTP layer
router.post('/profile/avatar', upload.single('avatar'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file provided' })
  }
  if (req.file.size > 5 * 1024 * 1024) {
    return res.status(400).json({ error: 'File too large' })
  }
  const result = await ProfileService.updateAvatar(req.auth.userId, req.file)
  res.json(result)
})

// services/ProfileService.ts — checks type at the service layer
class ProfileService {
  static async updateAvatar(userId: string, file: UploadedFile) {
    const mime = file.mimetype
    if (mime !== 'image/jpeg' && mime !== 'image/png') {
      throw new ValidationError('Only JPEG and PNG allowed')
    }
    const url = await StorageService.upload(file, `avatars/${userId}`)
    await db.users.update(userId, { avatarUrl: url })
    return { avatarUrl: url }
  }
}

// services/StorageService.ts — checks dimensions at the storage layer
class StorageService {
  static async upload(file: UploadedFile, path: string) {
    // "just in case" check since we're about to store it
    const dimensions = await sharp(file.buffer).metadata()
    if (dimensions.width < 100 || dimensions.height < 100) {
      throw new ValidationError('Image too small')
    }
    await s3.putObject({ Bucket: BUCKET, Key: path, Body: file.buffer })
    return `https://cdn.example.com/${path}`
  }
}
```

**Why this is wrong:**

The validation "works" in the sense that bad uploads get rejected. But:

1. **No single layer has the full picture.** If you ask "what are the avatar
   upload rules?", you have to read three files to answer. The rules are not
   documented in any one place — they are an emergent property of the call
   chain.

2. **Error messages are inconsistent.** The route returns 400 JSON for size
   errors. The service throws `ValidationError` for type errors. The storage
   layer throws `ValidationError` for dimension errors — but at that point,
   is the route's error handler going to produce a clean 400 response, or
   will it fall through to the 500 handler?

3. **Testing is fragmented.** To test all validation rules, you need to test
   through three different layers. Each layer's tests only verify its own
   partial check, so there is no single test that asserts the complete
   validation contract.

4. **`StorageService.upload` now has avatar-specific logic.** It was a generic
   upload utility. Now it knows about image dimensions. The next caller that
   uploads a PDF will either hit a confusing dimension check failure, or
   someone will add a parameter to skip the check — beginning the wrong
   abstraction spiral.

5. **The partial checks create false confidence.** A developer looking at the
   route sees size validation and assumes "validation is handled." They don't
   realize type checking happens in another file, and dimension checking in
   yet another. When a new rule is needed (e.g., "no animated GIFs"), it's
   unclear where to add it.

---

## Corrected Case — One Layer Owns the Complete Validation Concern

**Step 1 — Recognize the complexity placement problem:**

Validation for "what constitutes a valid avatar upload" is a single concern
that has been scattered across three layers. Each layer does a partial job,
and none of them owns the complete decision.

**Step 2 — Apply the judgment dimension (Complexity Placement):**

- Can this complexity be pulled into one place? → Yes. All three checks are
  about the same business rule: "is this a valid avatar?"
- Does scattering make callers simpler? → No. It makes the full validation
  contract invisible and forces readers to trace through three files.
- Is there a clear owning layer? → Yes. `ProfileService` owns avatar business
  logic. Validation of avatar inputs belongs there. `StorageService` should
  remain a generic upload utility.

**Step 3 — Decide: where should complexity live?**

`ProfileService.updateAvatar` owns the complete avatar validation contract.
The route stays thin (HTTP plumbing only). `StorageService` stays generic.

**Result:**
```typescript
// routes/profile.ts — thin HTTP plumbing, no business validation
router.post('/profile/avatar', upload.single('avatar'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file provided' })
  }
  const result = await ProfileService.updateAvatar(req.auth.userId, req.file)
  res.json(result)
})

// services/ProfileService.ts — owns avatar validation completely
class ProfileService {
  private static readonly AVATAR_RULES = {
    maxBytes: 5 * 1024 * 1024,
    allowedTypes: ['image/jpeg', 'image/png'] as const,
    minWidth: 100,
    minHeight: 100,
  }

  private static async validateAvatar(file: UploadedFile): Promise<void> {
    if (file.size > this.AVATAR_RULES.maxBytes) {
      throw new ValidationError('File too large (max 5MB)')
    }
    if (!this.AVATAR_RULES.allowedTypes.includes(file.mimetype)) {
      throw new ValidationError('Only JPEG and PNG images are allowed')
    }
    const { width, height } = await sharp(file.buffer).metadata()
    if (!width || !height || width < this.AVATAR_RULES.minWidth
        || height < this.AVATAR_RULES.minHeight) {
      throw new ValidationError('Image must be at least 100x100 pixels')
    }
  }

  static async updateAvatar(userId: string, file: UploadedFile) {
    await this.validateAvatar(file)
    const url = await StorageService.upload(file, `avatars/${userId}`)
    await db.users.update(userId, { avatarUrl: url })
    return { avatarUrl: url }
  }
}

// services/StorageService.ts — stays generic, no avatar knowledge
class StorageService {
  static async upload(file: UploadedFile, path: string) {
    await s3.putObject({ Bucket: BUCKET, Key: path, Body: file.buffer })
    return `https://cdn.example.com/${path}`
  }
}
```

All validation rules are in one place. The route only checks for "was a file
provided" (HTTP-layer concern). `StorageService` remains generic and reusable.
When a new avatar rule is needed, there is exactly one place to add it.

---

## The Distinction

| | Bad case | Corrected case |
|---|---|---|
| Invalid uploads are rejected | Yes | Yes |
| Validation rules visible in one place | No — scattered across 3 files | Yes — `ProfileService.validateAvatar` |
| Error handling is consistent | No — mixed 400 returns and thrown errors | Yes — one throw site, one catch site |
| StorageService stays generic | No — has avatar-specific dimension check | Yes — knows nothing about avatars |
| Adding a new rule is obvious | No — unclear which layer "should" own it | Yes — add to `validateAvatar` |
| Full validation is testable as a unit | No — must test through 3 layers | Yes — test `validateAvatar` directly |

"Defense in depth" is a valid security principle for network architecture.
But within a single application, it often becomes an excuse for scattered
ownership. When every layer does a partial job, no layer does the complete
job, and the result is harder to understand, test, and maintain than having
one layer own the concern completely.
