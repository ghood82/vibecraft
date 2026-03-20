# Evaluation Criteria Reference

Detailed rubrics, examples, and guidance for assessing code quality and outcomes.

## Detailed Rubrics by Dimension

### Correctness Rubric (1-5)

**5: Perfect Implementation**
- Handles all documented use cases
- Edge cases identified and handled
- Error states return meaningful feedback
- No known bugs or limitations
- Works in all tested scenarios
- Example: Form validation that handles empty, invalid, boundary values correctly

**4: Solid Implementation**
- Primary use case works perfectly
- Most edge cases handled
- Minor limitations documented
- Would need only bug fixes, not refactoring
- Example: API client that handles success and error cases, but doesn't handle network timeout retry

**3: Functional Implementation**
- Core functionality works
- Some edge cases unhandled
- Known limitations accepted
- Works for typical usage
- Example: Component that works in normal cases but has unexpected behavior with rapid user input

**2: Partially Working**
- Some scenarios work
- Common cases have issues
- Needs significant debugging
- Example: Search feature that works for simple queries but breaks with special characters

**1: Broken/Non-Functional**
- Doesn't work for intended use
- Fundamental logic error
- Needs complete rewrite
- Example: Authentication system that doesn't actually validate credentials

**Correctness Indicators:**
- Code tested against requirements? ✓/✗
- Error cases handled? ✓/✗
- Edge cases considered? ✓/✗
- Boundary conditions tested? ✓/✗
- Silent failures prevented? ✓/✗

### Completeness Rubric (1-5)

**5: Complete & Polished**
- All requested features implemented
- Documentation complete
- Tests comprehensive
- Error messages helpful
- Ready for production
- Example: Button component with all variants, states, sizes, accessibility, docs, and tests

**4: Complete Core**
- All primary features done
- Documentation adequate
- Basic tests present
- Minor polish possible
- Example: Button component with main variants and basic docs, some test gaps

**3: Core Functionality Done**
- Main features implemented
- Documentation minimal
- Tests partial
- Polish and optimization skipped
- Example: Button component with basic styling, limited documentation

**2: Substantially Incomplete**
- Major features missing
- Significant work remains
- Example: Button component skeleton, styling incomplete

**1: Barely Started**
- Minimal implementation
- Most work remains
- Example: Button component interface defined, implementation stub

**Completeness Indicators:**
- All features from spec implemented? ✓/✗
- Documentation written? ✓/✗
- Tests written? ✓/✗
- Error cases handled? ✓/✗
- Polish/refinement done? ✓/✗

### Code Quality Rubric (1-5)

**5: Excellent Code**
- Clear variable and function names
- Logical organization and structure
- DRY principle followed
- No magic numbers or strings
- Easy to understand at a glance
- Comments explain "why" not "what"
- Follows established patterns
- Example: Utility function with clear intent, nested logic abstracted into helpers, well-named variables

**4: Good Code**
- Generally clear and well-organized
- Minor style inconsistencies
- Could extract a helper or two
- Mostly follows patterns
- Example: Component with some nested logic that could be extracted but understandable as-is

**3: Acceptable Code**
- Works and understandable
- Some complexity or nesting
- Could be refactored
- Style inconsistencies present
- Example: Function that works but has deeply nested logic or unclear variable names

**2: Poor Code**
- Hard to follow
- Significant complexity
- Not following patterns
- Needs refactoring
- Example: Large function with mixed concerns, unclear variable names

**1: Terrible Code**
- Unreadable or dangerous
- No clear structure
- Contradicts best practices
- Example: Function with 15+ levels of nesting, cryptic variable names

**Code Quality Indicators:**
- Self-documenting code? ✓/✗
- Logical structure? ✓/✗
- DRY principle? ✓/✗
- Consistent style? ✓/✗
- Follows established patterns? ✓/✗

### Security Rubric (1-5)

**5: Security Best Practices**
- Input validated and sanitized
- Secrets protected (not in code)
- Safe defaults throughout
- HTTPS/encryption where needed
- No known vulnerabilities
- Example: API endpoint validates all inputs, returns generic errors, uses environment variables for secrets

**4: Mostly Secure**
- Main security concerns addressed
- Input validated
- No obvious vulnerabilities
- Minor hardening possible
- Example: API endpoint validates inputs, could use more defensive practices

**3: Acceptable Security**
- No obvious vulnerabilities
- Could be more defensive
- Basic validation present
- Example: API endpoint validates main cases, edge cases not considered

**2: Concerning Issues**
- Potential vulnerabilities identified
- Could be exploited with effort
- Needs security review
- Example: API endpoint validates inputs but doesn't sanitize, returns detailed error messages

**1: Risky/Dangerous**
- Serious vulnerabilities present
- Easily exploitable
- Example: API endpoint takes user input directly, no validation, executes it

**Security Indicators:**
- Input validated? ✓/✗
- Secrets protected? ✓/✗
- Safe error messages? ✓/✗
- Defense in depth? ✓/✗
- Vulnerability checked? ✓/✗

### UX/DX Rubric (1-5)

**5: Delightful Experience**
- Intuitive to use
- Fast and responsive
- Error messages clear and helpful
- Follows conventions
- Feels good to use
- Example: Form that prevents errors, provides live feedback, clear next steps

**4: Good Experience**
- Clear how to use
- Fast enough
- Error messages helpful
- Minor friction points
- Example: Form that works well, slightly unclear validation messages

**3: Acceptable Experience**
- Works but not obvious
- Some friction
- Error messages could help more
- Example: Form that works, but error messages are cryptic or confusing

**2: Frustrating**
- Confusing to use
- Slow or laggy
- Poor error messages
- Users get stuck
- Example: Form with unclear validation, cryptic errors

**1: Painful**
- Hard to figure out
- Very slow
- No guidance for users
- Example: Form with no feedback, unclear what's wrong

**UX/DX Indicators:**
- Obvious how to use? ✓/✗
- Error messages helpful? ✓/✗
- Responsive/fast? ✓/✗
- Follows conventions? ✓/✗
- Would I enjoy using this? ✓/✗

## Common Failure Modes & Root Causes

Understanding failure patterns helps prevent them.

### Logic Errors

**Pattern: Off-by-one errors**
- Symptom: Loop processes one item too many/few
- Root Cause: Confusion about inclusive vs. exclusive ranges
- Prevention: Write explicit range tests, use clear variable names (endIndex vs. count)
- Example Catch: `for (let i = 0; i <= array.length; i++)` should be `i < array.length`

**Pattern: Missing edge cases**
- Symptom: Works for normal input, fails with empty/null/special values
- Root Cause: Didn't consider all input types
- Prevention: Test with [], null, undefined, 0, empty string, boundary values
- Example Catch: `data.map(item => item.name)` breaks if item is null

**Pattern: Wrong condition logic**
- Symptom: Logic inverted or contradictory
- Root Cause: Complex boolean logic not walked through
- Prevention: Write out truth table, use De Morgan's laws
- Example Catch: `if (isActive && !isActive)` is impossible condition

### Oversight Errors

**Pattern: Missing validation**
- Symptom: Accepts invalid data silently or crashes
- Root Cause: Assumed input is always valid
- Prevention: Validate at boundaries, fail fast
- Example Catch: Function assumes user ID is number, but string passed

**Pattern: Missing error handling**
- Symptom: Unhandled exceptions crash the app
- Root Cause: Didn't consider what could fail
- Prevention: Try-catch around I/O, network, parsing
- Example Catch: JSON.parse without try-catch crashes on invalid JSON

**Pattern: Missing cleanup**
- Symptom: Timers, subscriptions, listeners leak
- Root Cause: Forgot to unsubscribe or unmount
- Prevention: Use useEffect cleanup, subscription patterns
- Example Catch: setInterval without clearInterval

### Assumption Errors

**Pattern: Wrong library behavior**
- Symptom: Code doesn't work as expected
- Root Cause: Misunderstood how library works
- Prevention: Test library behavior first, read docs
- Example Catch: Assumed `useEffect` runs before render, actually runs after

**Pattern: Wrong user context**
- Symptom: Built wrong feature or solved wrong problem
- Root Cause: Assumed user needs without confirming
- Prevention: Ask clarifying questions upfront
- Example Catch: Built dark mode toggle when user wanted theme switcher

**Pattern: Wrong data shape**
- Symptom: Code breaks because data is different
- Root Cause: Assumed API response shape
- Prevention: Log actual data, verify structure
- Example Catch: Assumed user.profile.name exists, but structure is nested differently

### Security Oversights

**Pattern: Input injection**
- Symptom: User input used unsafely (SQL, HTML, code injection)
- Root Cause: No input sanitization
- Prevention: Always validate and sanitize, use parameterized queries
- Example Catch: Directly interpolating user input into SQL query

**Pattern: Sensitive data exposure**
- Symptom: API keys, passwords, tokens in logs or code
- Root Cause: Used secrets in wrong place
- Prevention: Use environment variables, never log sensitive data
- Example Catch: API key hardcoded in client code

**Pattern: Missing authentication/authorization**
- Symptom: Unauthorized access to protected data
- Root Cause: Forgot to check permissions
- Prevention: Verify user is authenticated and authorized
- Example Catch: API endpoint doesn't check user role

## Before/After Examples of Improved Outputs

### Example 1: Component Structure

**Before (Score: 2)**
```typescript
export const UserProfile = ({ userId }) => {
  const [u, setU] = useState();
  const [l, setL] = useState([]);
  const [e, setE] = useState();
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(d => { setU(d); })
      .catch(er => { setE(er); });
    
    fetch(`/api/users/${userId}/likes`)
      .then(r => r.json())
      .then(d => { setL(d); })
      .catch(er => { setE(er); });
  }, [userId]);
  
  if (e) return <div>Error</div>;
  if (!u) return <div>Loading...</div>;
  
  return (
    <div>
      <h1>{u.name}</h1>
      <p>{u.bio}</p>
      <div>{l.map(x => <div key={x.id}>{x.title}</div>)}</div>
    </div>
  );
};
```

Issues: Cryptic variable names, mixed concerns, poor error messages

**After (Score: 5)**
```typescript
// Extracted hooks
function useUserProfile(userId: string) {
  const [user, setUser] = useState<User | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  
  useEffect(() => {
    fetchUserProfile(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setIsLoading(false));
  }, [userId]);
  
  return { user, error, isLoading };
}

// Clear, focused component
export const UserProfile = ({ userId }: UserProfileProps) => {
  const { user, error, isLoading } = useUserProfile(userId);
  
  if (isLoading) return <UserProfileSkeleton />;
  if (error) return <ErrorBoundary error={error} />;
  if (!user) return <div>User not found</div>;
  
  return (
    <article>
      <UserHeader user={user} />
      <UserBio user={user} />
      <UserLikes userId={userId} />
    </article>
  );
};
```

Improvements: Clear names, separated concerns, proper error handling, reusable hooks

### Example 2: Error Handling

**Before (Score: 2)**
```typescript
async function submitForm(data) {
  const res = await fetch('/api/submit', { method: 'POST', body: JSON.stringify(data) });
  return res.json();
}
```

Issues: No error handling, assumes success, cryptic errors

**After (Score: 5)**
```typescript
class FormError extends Error {
  constructor(
    public code: string,
    public userMessage: string,
    public details?: unknown
  ) {
    super(userMessage);
    this.name = 'FormError';
  }
}

async function submitForm(data: FormData): Promise<SubmitResponse> {
  try {
    // Validate before submission
    validateFormData(data);
    
    const response = await fetch('/api/submit', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    
    if (!response.ok) {
      const errorData = await response.json();
      throw new FormError(
        errorData.code || 'SUBMIT_FAILED',
        errorData.message || 'Failed to submit form. Please try again.',
        { status: response.status }
      );
    }
    
    return response.json();
  } catch (error) {
    if (error instanceof FormError) {
      throw error; // Re-throw known errors
    }
    
    // Handle unknown errors
    throw new FormError(
      'UNEXPECTED_ERROR',
      'An unexpected error occurred. Please try again later.',
      { originalError: error }
    );
  }
}
```

Improvements: Input validation, typed errors, meaningful messages, safe error handling

## Metrics to Track Over Time

Build a dashboard of your performance trends.

### Code Quality Metrics

| Metric | Measurement | Target | Notes |
|--------|-------------|--------|-------|
| Average Correctness | Mean of 1-5 scores | 4.2+ | Indicates reliability |
| Average Code Quality | Mean of 1-5 scores | 4.0+ | Indicates maintainability |
| Bug Discovery Rate | Bugs found in testing | <1 per 5 tasks | Earlier detection better |
| Rework Rate | % of tasks needing revision | <15% | Low rework = good planning |
| Completeness % | Features built / requested | >95% | Few missing features |

### Learning Metrics

| Metric | Measurement | Target | Notes |
|--------|-------------|--------|-------|
| Patterns Documented | New patterns per month | 2-4 | Captures what you learn |
| Mistake Recurrence | Same mistake twice | 0 | Learning is working |
| Session Retrospectives | Completed per month | 4-8 | Regular reflection |
| Memory Promotions | Items added to CLAUDE.md | 1-2 per month | Core context growing |
| Skill Improvements | Suggestions implemented | 2-4 per quarter | Continuous improvement |

### Speed Metrics

| Metric | Measurement | Target | Notes |
|--------|-------------|--------|-------|
| Iteration Time | Sessions to feature complete | Decreasing | Getting faster |
| Decision Speed | Time to make arch decisions | Decreasing | Better pattern reuse |
| Debug Time | Time to find/fix bugs | Decreasing | Patterns prevent bugs |

## Self-Improvement Loop Protocol

Structured process for continuous improvement.

### Weekly Review (15 min)
1. List all tasks from the week
2. Score each on 5 dimensions
3. Identify 1 success pattern
4. Identify 1 failure pattern
5. Document 1 learning

### Monthly Reflection (30 min)
1. Review weekly reviews
2. Calculate average scores per dimension
3. Identify trends (improving/declining)
4. Promote 1-2 patterns to memory/
5. Archive 1-2 outdated patterns
6. Suggest 1 skill improvement

### Quarterly Review (1 hour)
1. Review all metrics
2. Identify strongest and weakest dimensions
3. Set improvement targets
4. Update CLAUDE.md with learnings
5. Plan skill enhancements
6. Celebrate improvements

---

**Start Here**: Track next 5 tasks with the evaluation rubric. Calculate averages. Notice patterns.
