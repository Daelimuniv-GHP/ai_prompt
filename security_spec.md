# Security Specification (`prompts` Collection)

## 1. Data Invariants
- Every document in `/prompts/{promptId}` must have a valid `promptId` matching `^[a-zA-Z0-9_\-]+$` (max 128 chars).
- Every prompt document must contain strictly the 4 fields: `title`, `category`, `content`, and `createdAt` (no shadow fields).
- `title` must be a non-empty string with length between 1 and 100 characters.
- `category` must strictly be one of `'사진 합성'`, `'스타일 변환'`, or `'캐릭터·포스터'`.
- `content` must be a non-empty string with length between 1 and 10,000 characters (supporting long prompts over 1,000 characters while preventing abuse).
- `createdAt` must equal `request.time` on creation and remain immutable on update.
- **Public Read / Admin-Only Write**: Anyone (even without logging in) can read `/prompts/{promptId}`. Only the verified administrator (`request.auth != null && request.auth.token.email == 'gihunbag02@gmail.com' && request.auth.token.email_verified == true`) can create, update, or delete prompts.
- All other collections are completely locked down (`allow read, write: if false`).

## 2. Adversarial Payloads ("Dirty Dozen")
1. Unauthenticated `create` / `update` / `delete` -> `PERMISSION_DENIED`
2. Non-admin authenticated `create` / `update` / `delete` -> `PERMISSION_DENIED`
3. Spoofed admin email with `email_verified == false` -> `PERMISSION_DENIED`
4. Missing `title` field -> `PERMISSION_DENIED`
5. Empty `title` (`""`) -> `PERMISSION_DENIED`
6. Oversized `title` (> 100 chars) -> `PERMISSION_DENIED`
7. Invalid `category` (`"기타"`) -> `PERMISSION_DENIED`
8. Empty `content` (`""`) -> `PERMISSION_DENIED`
9. Oversized `content` (> 10,000 chars) -> `PERMISSION_DENIED`
10. Spoofed `createdAt` (client timestamp != `request.time`) -> `PERMISSION_DENIED`
11. Shadow field injection (`{ ..., isAdmin: true }`) -> `PERMISSION_DENIED`
12. Access to any collection other than `prompts` -> `PERMISSION_DENIED`
