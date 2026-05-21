# Complexity Misplacement - One Concern, One Owner

Read this when validation, transformation, recovery, authorization, or a domain
rule appears in multiple layers, or when a patch would make several layers each
handle one part of the same concern.

Good code keeps a complete concern in the layer that has enough context to own
the decision. A patch is not structurally honest just because the behavior works
on the current path.

---

## Scenario

**Task:** "Add avatar upload validation: max 5MB, JPEG/PNG only, minimum
100x100 pixels."

**Current flow:**

```text
routes/profile.ts -> ProfileService.updateAvatar -> StorageService.upload
```

The route owns HTTP plumbing. `ProfileService` owns avatar business behavior.
`StorageService` is a generic upload primitive.

---

## Bad Move - Split One Validation Contract Across Layers

```typescript
// routes/profile.ts
if (req.file.size > MAX_AVATAR_BYTES) return res.status(400).json(...)

// ProfileService.ts
if (!ALLOWED_AVATAR_TYPES.includes(file.mimetype)) throw new ValidationError(...)

// StorageService.ts
if ((await imageSize(file)).width < 100) throw new ValidationError(...)
```

This can reject invalid uploads, but the ownership is wrong:

- The complete avatar rule is only visible after reading three files.
- Error behavior now depends on which partial check failed.
- Tests can pass for each partial check without proving the full avatar
  validation contract.
- `StorageService` now knows avatar-specific image rules, so the next non-avatar
  upload can inherit irrelevant behavior or force a caller-specific flag.

---

## Judgment to Apply

When a concern is being split across layers:

- Name the complete concern in one sentence, such as "valid avatar upload."
- List each layer that reads, validates, transforms, stores, or recovers that
  concern.
- Keep boundary-only checks at the boundary, such as "was a file provided" or
  "can this request be parsed."
- Put domain rules in the layer with domain context. In this case,
  `ProfileService` owns avatar rules.
- Keep generic infrastructure generic. `StorageService.upload` may validate
  storage-level invariants, but it should not know avatar dimensions.
- If no layer has enough context to own the complete decision, pause and surface
  the missing ownership boundary before coding.

Correct shape:

```typescript
// routes/profile.ts
if (!req.file) return res.status(400).json(...)
await ProfileService.updateAvatar(req.auth.userId, req.file)

// ProfileService.ts
await validateAvatar(file) // size, type, dimensions live together
await StorageService.upload(file, `avatars/${userId}`)

// StorageService.ts
await s3.putObject(...)
```

---

## Risk Signals

Treat these as evidence that complexity is misplaced:

- You must trace multiple layers to answer "what is the full rule?"
- Different parts of one rule return errors through different mechanisms.
- A generic utility imports or names domain concepts.
- Adding the next rule has no obvious file.
- Testing the full rule requires unrelated route, service, and infrastructure
  tests instead of one owner-level test.

Do not use "defense in depth" to justify scattered ownership inside one
application. Defense in depth is useful when independent boundaries each enforce
their own rule. It is harmful when several layers each own a fragment of one
rule.
