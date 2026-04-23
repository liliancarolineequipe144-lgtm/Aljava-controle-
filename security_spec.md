# Security Specification - Aljava Controle

## 1. Data Invariants
- A `child` must belong to a `familyId` or have a `parentId`.
- `checkedIn` status can only be toggled by an authorized volunteer or admin.
- `volunteers` can only be created/modified by an admin (Coordenador).
- `schedules` and `services` are restricted to admin modification.
- `settings` is global read but write-restricted to admin.

## 2. The Dirty Dozen Payloads (Target: permission_denied)

### Payload 1: Unauthenticated Write (Child)
- **Path**: `/children/malicious_child`
- **Auth**: `null`
- **Data**: `{ "name": "Hackie", "birthDate": "2020-01-01" }`
- **Goal**: Prevent random internet users from adding children.

### Payload 2: Self-Promotion (Volunteer)
- **Path**: `/volunteers/my_uid`
- **Auth**: `{ "uid": "my_uid", "token": { "email_verified": true } }`
- **Data**: `{ "name": "Me", "phone": "123", "role": "Coordenador", "active": true }`
- **Goal**: Prevent users from making themselves coordinators.

### Payload 3: ID Poisoning (Service)
- **Path**: `/services/A_VERY_LONG_ID_EXCEEDING_128_CHARS_TO_EXHAUST_RESOURCES_AND_TEST_GAURDS_..._JUNK`
- **Auth**: admin
- **Data**: `{ "name": "Service", "date": "2024-01-01", "type": "Culto" }`
- **Goal**: Block resource exhaustion via massive IDs.

### Payload 4: Ghost Field Injection (Settings)
- **Path**: `/settings/app`
- **Auth**: admin
- **Data**: `{ "logoUrl": "...", "appName": "App", "maliciousField": "HIDDEN_VALUE" }`
- **Goal**: Ensure strict schema adherence via `affectedKeys().hasOnly()`.

### Payload 5: Identity Spoofing (Parent)
- **Path**: `/parents/target_parent`
- **Auth**: `{ "uid": "attacker_uid" }`
- **Data**: `{ "name": "Spoof", "phone": "999", "leader": "L", "familyId": "victim_family" }`
- **Goal**: Prevent users from modifying other parents' data.

### Payload 6: Status Shortcutting (Child Check-in)
- **Path**: `/children/victim_child`
- **Auth**: normal_user
- **Data**: update `{ "checkedIn": true }`
- **Goal**: Only authorized roles should toggle check-in.

### Payload 7: Timestamp Manipulation (Material)
- **Path**: `/materials/m1`
- **Auth**: admin
- **Data**: update `{ "quantity": 10, "lastUpdated": "2000-01-01T00:00:00Z" }`
- **Goal**: Enforce server timestamps `request.time`.

### Payload 8: PII Blanket Read (Parent)
- **Operation**: `list` on `/parents`
- **Auth**: `{ "uid": "random_user" }`
- **Goal**: Prevent non-admins from listing all parent phones and data.

### Payload 9: Orphaned Record (Child)
- **Path**: `/children/c1`
- **Auth**: admin
- **Data**: `{ "name": "Child", "birthDate": "2020", "familyId": "NON_EXISTENT_FAMILY" }`
- **Goal**: (Optional but good) Ensure relational integrity.

### Payload 10: Terminal State Violation (Service)
- **Path**: `/services/s1`
- **Auth**: admin
- **Data**: update `{ "date": "2025-01-01" }`
- **State**: `s1` is already in a past/terminal date.
- **Goal**: Prevent modification of past service records if applicable.

### Payload 11: Negative Resource (Material)
- **Path**: `/materials/m1`
- **Auth**: admin
- **Data**: `{ "name": "Paper", "quantity": -5, "category": "Office" }`
- **Goal**: Boundary checks on numeric fields.

### Payload 12: Unauthorized Delete (Schedule)
- **Path**: `/schedules/s1`
- **Auth**: normal_user
- **Goal**: Prevent non-admins from deleting schedules.

## 3. Test Runner (Mockup for firestore.rules.test.ts)
- Implement tests verifying `PERMISSION_DENIED` for all above.
