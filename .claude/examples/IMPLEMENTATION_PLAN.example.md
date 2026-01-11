# Example: IMPLEMENTATION_PLAN.md

**Issue:** add-oauth-login
**Feature:** Add OAuth2 authentication with Google and GitHub providers

---

### Overview

**Goal:** Add Google OAuth2 authentication to complement existing GitHub OAuth, following established patterns

**Success Criteria:**
- [ ] Users can authenticate via Google OAuth2
- [ ] Session tracks which OAuth provider was used
- [ ] OAuth failures are handled gracefully with user feedback
- [ ] All tests pass (unit + integration)
- [ ] Existing GitHub OAuth functionality unaffected
- [ ] Code follows existing patterns and conventions

**Out of Scope:**
- Token refresh implementation (deferred to future iteration)
- Database storage for OAuth tokens (using session storage)
- Additional OAuth providers (just Google + GitHub)

---

### Phases

**Phase 1: Foundation** (Complexity: Low, Est: 30min)
- `src/auth/google-strategy.ts:1-60` - Create Google OAuth2 strategy
  - Configure strategy with environment variables
  - Implement profile extraction
  - Implement user lookup/creation callback
  - Follow pattern from `src/auth/github-strategy.ts`
- `src/auth/passport.ts:50-70` - Register Google strategy with passport
  - Import GoogleStrategy
  - Initialize with credentials
  - Add to passport middleware
- `package.json:25` - Add passport-google-oauth20 dependency

**Phase 2: Routes & Controllers** (Complexity: Medium, Est: 1hr)
- `src/routes/auth.ts:45-80` - Add Google OAuth routes
  - GET /auth/google - Initiate Google OAuth flow
  - GET /auth/google/callback - Handle Google OAuth callback
  - Follow pattern from GitHub routes
- `src/controllers/auth.ts:100-150` - Add Google OAuth callback handler
  - Extract user profile from Google
  - Create or update user record
  - Set session with provider tracking
  - Handle OAuth errors
- `src/types/auth.ts:20-40` - Add OAuth provider types
  - Add `OAuthProvider` type ('google' | 'github')
  - Extend session interface with provider field

**Phase 3: Session Management** (Complexity: Medium, Est: 45min)
- `src/middleware/session.ts:120-150` - Update session for provider tracking
  - Add `provider` field to session object
  - Update session helpers to include provider
  - Ensure backward compatibility (GitHub sessions without provider)
- `src/middleware/auth.ts:50-75` - Update auth middleware
  - Handle sessions with provider field
  - Maintain compatibility for existing sessions

**Phase 4: Testing** (Complexity: Medium, Est: 1.5hr)
- `tests/mocks/oauth.ts:1-50` - Create OAuth provider mocks (NEW FILE)
  - Mock Google OAuth profile
  - Mock passport strategy
  - Helper functions for mock OAuth flows
- `tests/auth/oauth.test.ts:1-100` - Add OAuth flow tests (NEW FILE)
  - Test Google OAuth initiate
  - Test Google OAuth callback success
  - Test Google OAuth callback failure
  - Test session provider tracking
  - Test GitHub OAuth still works
- `tests/integration/auth.test.ts:50-100` - Add integration tests
  - Test full OAuth flow with mocks
  - Test error scenarios
  - Test session management

**Phase 5: Polish & Documentation** (Complexity: Low, Est: 30min)
- `docs/auth/oauth.md:1-100` - Document OAuth setup (NEW FILE)
  - Google OAuth setup instructions
  - Environment variable configuration
  - Troubleshooting common issues
- `.env.example:5-6` - Add Google OAuth env var examples
  - GOOGLE_CLIENT_ID
  - GOOGLE_CLIENT_SECRET
- Verify all tests pass: `npm test`
- Verify type check passes: `npm run type-check`
- Verify linter passes: `npm run lint`

---

### Testing Strategy

**Unit Tests:**
- GoogleOAuthStrategy configuration
- Profile extraction from Google
- User lookup/creation callback
- Session provider tracking
- Error handling for OAuth failures

**Integration Tests:**
- Full OAuth flow (initiate → callback → session)
- OAuth failure scenarios (invalid token, network error)
- Session management with provider field
- Backward compatibility with GitHub OAuth

**Edge Cases:**
- OAuth callback with error parameter
- User denies OAuth consent
- Network timeout during OAuth flow
- Invalid OAuth state parameter
- Session expired before callback

**Testing Requirements:**
- Mock passport strategies (no real OAuth calls)
- Mock user database operations
- Test error scenarios explicitly
- Maintain 90%+ code coverage for new code

---

### Estimates

| Phase | Complexity | Estimated Time | Dependencies |
| ----- | ---------- | -------------- | ------------ |
| 1     | Low        | 30min          | None |
| 2     | Medium     | 1hr            | Phase 1 complete |
| 3     | Medium     | 45min          | Phase 2 complete |
| 4     | Medium     | 1.5hr          | Phase 3 complete |
| 5     | Low        | 30min          | Phase 4 complete |
| **Total** | | **4 hours** | |

---

### Dependencies

**External Packages:**
```json
{
  "passport-google-oauth20": "^2.0.0"
}
```

**Dev Dependencies:**
```json
{
  "@types/passport-google-oauth20": "^2.0.0"
}
```

**Environment Variables:**
```bash
GOOGLE_CLIENT_ID=your-google-client-id-from-console
GOOGLE_CLIENT_SECRET=your-google-client-secret-from-console
GOOGLE_CALLBACK_URL=http://localhost:3000/auth/google/callback
```

**Existing Code Dependencies:**
- `src/auth/github-strategy.ts` - Reference pattern
- `src/middleware/session.ts` - Extend for provider tracking
- `src/types/auth.ts` - Add provider types

---

### File-by-File Breakdown

**NEW FILES:**
```
src/auth/google-strategy.ts
├── Lines 1-10: Imports (GoogleStrategy, User model, etc.)
├── Lines 11-30: Strategy configuration (clientID, secret, callbackURL)
├── Lines 31-50: Verify callback (find/create user)
└── Lines 51-60: Export strategy

tests/mocks/oauth.ts
├── Lines 1-20: Mock Google profile
├── Lines 21-40: Mock passport strategy
└── Lines 41-50: Helper functions

tests/auth/oauth.test.ts
├── Lines 1-20: Test setup (mocks, fixtures)
├── Lines 21-50: Strategy configuration tests
├── Lines 51-80: OAuth flow tests
└── Lines 81-100: Error scenario tests

docs/auth/oauth.md
├── Lines 1-30: Google OAuth setup
├── Lines 31-60: Environment configuration
└── Lines 61-100: Troubleshooting
```

**MODIFIED FILES:**
```
src/auth/passport.ts:50-70
├── Lines 50-55: Import GoogleStrategy
├── Lines 56-65: Configure Google strategy
└── Lines 66-70: Register with passport

src/routes/auth.ts:45-80
├── Lines 45-55: GET /auth/google route
├── Lines 56-70: GET /auth/google/callback route
└── Lines 71-80: Error handling

src/controllers/auth.ts:100-150
├── Lines 100-120: Google OAuth callback handler
├── Lines 121-135: User creation/update logic
├── Lines 136-145: Session setup with provider
└── Lines 146-150: Error handling

src/types/auth.ts:20-40
├── Lines 20-30: OAuthProvider type definition
└── Lines 31-40: Session interface extension

src/middleware/session.ts:120-150
├── Lines 120-130: Add provider to session object
├── Lines 131-140: Update session helpers
└── Lines 141-150: Backward compatibility

package.json:25
└── Line 25: Add passport-google-oauth20 dependency

.env.example:5-6
├── Line 5: GOOGLE_CLIENT_ID example
└── Line 6: GOOGLE_CLIENT_SECRET example
```

---

### Success Validation

**After implementation, verify:**

1. **Authentication Works:**
   - [ ] Can click "Sign in with Google"
   - [ ] Redirected to Google OAuth consent screen
   - [ ] After consent, redirected back to app
   - [ ] User is logged in with session
   - [ ] Session shows provider: 'google'

2. **GitHub OAuth Still Works:**
   - [ ] Can still sign in with GitHub
   - [ ] Session shows provider: 'github'
   - [ ] No breaking changes to existing flow

3. **Error Handling:**
   - [ ] OAuth denial shows user-friendly error
   - [ ] Network errors handled gracefully
   - [ ] Invalid tokens handled appropriately
   - [ ] Error pages redirect correctly

4. **Tests Pass:**
   - [ ] Unit tests: 100% passing
   - [ ] Integration tests: 100% passing
   - [ ] Coverage: 90%+ for new code
   - [ ] No regressions in existing tests

5. **Code Quality:**
   - [ ] Linter passes (0 errors, 0 warnings)
   - [ ] Type check passes (0 errors)
   - [ ] Code follows existing patterns
   - [ ] Comments where needed

---

**End of Implementation Plan**
