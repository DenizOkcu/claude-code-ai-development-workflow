# Example: CODE_RESEARCH.md

**Issue:** add-oauth-login
**Feature Description:** Add OAuth2 authentication with Google and GitHub providers

---

### Summary

- **Risk:** Medium
- **Integration Complexity:** Medium
- **Key Findings:**
  - Existing auth system uses passport.js in `src/auth/`
  - Session management already handles user sessions
  - GitHub OAuth strategy already implemented
  - Need to add Google OAuth2 strategy
  - Session storage needs provider tracking
- **Top Recommendations:**
  - Reuse existing passport.js configuration
  - Follow GitHub OAuth pattern for Google
  - Update session handling to track OAuth provider
  - Mock OAuth providers for testing
- **Questions to Answer:**
  - Should OAuth tokens be stored in database or session?
  - Do we need token refresh logic?
  - How to handle OAuth failure scenarios?

---

### Integration Points

**Files to Modify:**
- `src/auth/passport.ts:50-100` - Add Google OAuth2 strategy configuration
- `src/middleware/session.ts:120-150` - Update session to include provider field
- `src/routes/auth.ts:45-80` - Add Google OAuth routes (/auth/google, /auth/google/callback)
- `src/types/auth.ts:20-40` - Add OAuth provider types
- `package.json:1` - Add passport-google-oauth20 dependency

**Files to Create:**
- `src/auth/google-strategy.ts` - Google OAuth2 strategy implementation (NEW)
- `src/controllers/auth.ts:100-150` - Google OAuth callback handler (NEW or modify)

**Reusable Patterns:**
- `src/auth/github-strategy.ts:1-60` - Reference implementation for OAuth strategy
- `src/middleware/auth.ts:20-50` - Session management pattern to follow
- `tests/mocks/github.ts:1-40` - Mock OAuth provider pattern

**Dependencies to Add:**
- `passport-google-oauth20` - Google OAuth2 strategy for passport.js

**Dependencies Already Present:**
- `passport` - Authentication middleware (already used)
- `passport-github2` - GitHub OAuth (already implemented)
- `express-session` - Session management (already used)
- `@types/passport` - TypeScript types (already configured)

---

### Technical Context

**Stack:**
- **Runtime:** Node.js 18+
- **Framework:** Express.js 4.x
- **Language:** TypeScript 5.x (strict mode)
- **Authentication:** passport.js 0.6+
- **Session:** express-session with memory store (development), Redis (production)

**Architecture Pattern:**
- MVC (Model-View-Controller)
- Middleware-based request handling
- Service layer for business logic
- Route handlers → Controllers → Services

**Communication Patterns:**
- Callback-based async for passport.js
- Promise-based async for services
- Event-driven for session updates
- REST API endpoints

**Code Conventions:**
- **Naming:** camelCase for variables, PascalCase for classes/types
- **File Structure:** Feature-based (auth/, routes/, controllers/, services/)
- **Imports:** Absolute imports from `src/` root
- **Type Safety:** Strict TypeScript, no `any` types without justification
- **Error Handling:** Try-catch with specific error types, custom error classes
- **Testing:** Jest with mocks, integration tests for API endpoints

**Authentication Flow (Current):**
```
User → POST /auth/github → GitHub OAuth → Callback → Session Created
```

**Authentication Flow (After Changes):**
```
User → POST /auth/google OR /auth/github → OAuth → Callback → Session Updated
```

---

### Risks

**Medium Risk:**
- **Session Size Increase**
  - Impact: Storing multiple OAuth tokens in session increases memory usage
  - Mitigation: Consider token storage in database for production, session for development
  - Monitoring: Check session size after implementation

- **Token Security**
  - Impact: OAuth tokens stored in session (memory store) not encrypted
  - Mitigation: Use Redis with encryption for production
  - Documentation: Document security implications

**Low Risk:**
- **OAuth Provider Limits**
  - Impact: Google OAuth has rate limits (10,000 requests/day default)
  - Mitigation: Not an issue for typical usage, monitor if needed
  - Reference: [Google OAuth Limits](https://developers.google.com/identity/protocols/oauth2)

- **Token Expiry**
  - Impact: Access tokens expire (typically 1 hour)
  - Mitigation: Implement token refresh if long-lived access needed
  - Current: Session-based auth (tokens live as long as session)

**Very Low Risk:**
- **Dependency Version Conflicts**
  - Impact: passport-google-oauth20 version compatibility
  - Mitigation: Use latest stable version, test thoroughly
  - Current: passport@0.6.0, should be compatible

---

### Key Files

```
src/auth/passport.ts (200 lines)
├── 1-50: Imports and passport initialization
├── 51-100: GitHub OAuth strategy configuration ← ADD Google strategy here
├── 101-150: Serialization/deserialization functions
└── 151-200: Export passport instance

src/auth/github-strategy.ts (80 lines) ← REFERENCE for Google strategy
├── 1-30: GitHub OAuth strategy configuration
├── 31-60: Profile extraction and user lookup
└── 61-80: Strategy callback function

src/middleware/session.ts (150 lines)
├── 1-50: Session configuration
├── 51-100: Session middleware setup
├── 101-120: User session helpers ← ADD provider tracking here
└── 121-150: Session validation

src/routes/auth.ts (100 lines)
├── 1-40: GitHub OAuth routes ← ADD Google routes here
├── 41-80: Callback handlers
└── 81-100: Logout route

src/types/auth.ts (50 lines) ← EXTEND for provider types
├── 1-20: User type definitions
├── 21-40: Session types ← ADD OAuth provider types
└── 41-50: Auth-related interfaces
```

---

### Similar Functionality Analysis

**GitHub OAuth Implementation (Reference):**
- **Location:** `src/auth/github-strategy.ts`
- **Pattern:**
  1. Configure strategy with client ID/secret
  2. Define profile callback (extract user info)
  3. Define verify callback (find/create user)
  4. Register strategy with passport

**Key Code from GitHub Strategy:**
```typescript
// src/auth/github-strategy.ts:20-40
new GitHubStrategy({
    clientID: process.env.GITHUB_CLIENT_ID,
    callbackURL: "/auth/github/callback"
  },
  async (accessToken, refreshToken, profile, done) => {
    // Find or create user
    // Return user object
  }
);
```

**Apply This Pattern for Google:**
- Use `GoogleStrategy` from `passport-google-oauth20`
- Same callback structure
- Extract different profile fields (Google vs GitHub)

---

### Open Questions

1. **Token Storage Strategy**
   - **Question:** Should OAuth access tokens be stored in database or session?
   - **Current:** Session storage (follows existing pattern)
   - **Consideration:** Database storage would support token refresh and multi-device
   - **Decision Needed:** For MVP, session storage is acceptable
   - **Future Consideration:** Migrate to database if token refresh needed

2. **Token Refresh Implementation**
   - **Question:** Do we need to implement token refresh logic?
   - **Current:** No refresh (tokens live as long as session)
   - **Consideration:** Google tokens expire after 1 hour
   - **Decision Needed:** Only if long-lived API access is required
   - **Impact:** If not implemented, users may need to re-auth after expiry

3. **OAuth Failure Handling**
   - **Question:** How should OAuth failures be handled?
   - **Options:**
     - Show error page with retry button
     - Redirect to login page with error message
     - Log error and show generic error
   - **Current:** GitHub implementation uses error redirect
   - **Recommendation:** Follow GitHub pattern for consistency

4. **Provider Display in UI**
   - **Question:** Should UI show which OAuth provider user used?
   - **Options:**
     - Show provider icon (Google/GitHub logo)
     - Show provider name in text
     - No indication (just "Logged in")
   - **Current:** Session doesn't track provider
   - **Decision Needed:** Add `provider: 'google' | 'github'` to session

5. **Testing Strategy for OAuth**
   - **Question:** How to test OAuth flows without real OAuth providers?
   - **Current:** GitHub has mocks in `tests/mocks/github.ts`
   - **Recommendation:** Create similar mocks for Google OAuth
   - **Implementation:** Mock passport strategy, test callback handlers

---

## Notes for Planning Phase

**What to Include in IMPLEMENTATION_PLAN.md:**
- Phase 1: Foundation (Google strategy setup, passport config)
- Phase 2: Routes & Controllers (Google OAuth routes, callbacks)
- Phase 3: Session Updates (provider tracking, session types)
- Phase 4: Testing (mocks, integration tests, error scenarios)
- Phase 5: Documentation (env vars, setup instructions)

**Dependencies to Install:**
```bash
npm install passport-google-oauth20
npm install --save-dev @types/passport-google-oauth20
```

**Environment Variables Needed:**
```bash
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret
GOOGLE_CALLBACK_URL=http://localhost:3000/auth/google/callback
```

**Success Criteria:**
- Users can authenticate via Google OAuth2
- Session tracks which provider was used
- OAuth failures are handled gracefully
- Tests mock OAuth providers
- All existing GitHub OAuth functionality still works

---

**End of Research Document**
