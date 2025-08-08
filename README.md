# Development Guidelines

## Philosophy

### Core Beliefs

- **Incremental progress over big bangs** - Small, atomic changes that compile and pass tests
- **Learning from existing code** - Study, understand patterns, and plan before implementing
- **Pragmatic over dogmatic** - Adapt to project reality while maintaining standards
- **Clear intent over clever code** - Be boring, obvious, and maintainable
- **Data-driven decisions** - Measure performance impacts, don't assume
- **Fail fast, recover faster** - Early validation and graceful degradation

### Simplicity Manifesto

- Single responsibility per function/class (max 30 lines per function as guideline)
- Avoid premature abstractions - need it 3 times before abstracting
- No clever tricks - choose the boring solution
- If you need to explain it, it's too complex
- Delete code ruthlessly - less code = less bugs

## Process

### 1. Planning & Staging

Break complex work into 3-5 stages. Document in `IMPLEMENTATION_PLAN.md`:

```markdown
## Stage N: [Name]

**Goal**: [Specific deliverable]
**Success Criteria**: [Testable outcomes]
**Tests**: [Specific test cases]
**Dependencies**: [What must exist before this]
**Risks**: [What could go wrong]
**Rollback Plan**: [How to undo if needed]
**Status**: [Not Started|In Progress|Complete|Blocked]
**Time Estimate**: [Hours/Days]
**Actual Time**: [Track for learning]
```

Update status as you progress. Remove file when all stages complete.

### 2. Implementation Flow

1. **Understand**

   - Study existing patterns in codebase
   - Find 3 similar implementations
   - Document assumptions in PR/commit

2. **Design**

   - Write interface/API first
   - Get early feedback on approach
   - Consider edge cases upfront

3. **Test**

   - Write test first (red)
   - Include edge cases
   - Performance benchmarks if relevant

4. **Implement**

   - Minimal code to pass (green)
   - Follow existing patterns
   - Add telemetry/logging

5. **Refactor**

   - Clean up with tests passing
   - Optimize only with data
   - Update documentation

6. **Review**

   - Self-review with fresh eyes
   - Run through checklist
   - Test manually once

7. **Commit**
   - Atomic commits with clear messages
   - Link to issue/plan
   - Include "why" not just "what"

### 3. When Stuck Protocol

**CRITICAL**: Maximum 3 attempts per issue, then STOP.

#### After Each Failed Attempt:

1. **Document precisely**:

   ```markdown
   ## Attempt N

   **Hypothesis**: What I thought would work
   **Implementation**: What I actually did
   **Result**: Exact error/failure
   **Learning**: Why it didn't work
   ```

2. **After 3 Attempts - MANDATORY STOP**:

   - Take 15-minute break minimum
   - Write up findings in team channel/issue
   - Consider pair programming
   - Question if solving right problem

3. **Research alternatives**:

   - Find 2-3 similar implementations
   - Check Stack Overflow with exact error
   - Review framework/library docs
   - Look for existing issues

4. **Question fundamentals**:
   - Is this the right abstraction level?
   - Can this be split into smaller problems?
   - Is there a simpler approach entirely?
   - Am I over-engineering?

## Technical Standards

### Architecture Principles

- **Composition over inheritance** - Use dependency injection and interfaces
- **Explicit over implicit** - Clear data flow, no magic
- **Immutability by default** - Mutate only when necessary with clear ownership
- **Async by design** - Non-blocking operations, proper cancellation
- **Cache invalidation strategy** - Define upfront, not afterthought
- **Feature flags for risky changes** - Deploy != Release

### Code Quality Metrics

- **Every commit must**:

  - [ ] Compile successfully
  - [ ] Pass all existing tests
  - [ ] Include tests for new functionality (aim for 80% coverage)
  - [ ] Pass linting/formatting
  - [ ] Update relevant documentation
  - [ ] Not degrade performance by >5%

- **Before committing**:
  ```bash
  # Run this checklist
  make lint
  make test
  make bench  # if performance sensitive
  git diff --staged  # self-review
  ```

### Error Handling Strategy

```typescript
// Example pattern
class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public statusCode: number,
    public isOperational: boolean = true
  ) {
    super(message);
    Error.captureStackTrace(this, this.constructor);
  }
}
```

- Fail fast with descriptive messages
- Include context for debugging (request ID, user ID, etc.)
- Distinguish operational vs programming errors
- Log errors with appropriate severity
- Never silently swallow exceptions
- Implement circuit breakers for external services

### Performance Guidelines

- **Measure before optimizing** - Use profilers, not intuition
- **Set performance budgets**:
  - API response: <200ms p50, <1s p99
  - Frontend interaction: <100ms
  - Build time: <2 minutes
- **Monitor in production**:
  - APM tools required
  - Real user metrics (RUM)
  - Set up alerts for degradation

## Decision Framework

When multiple valid approaches exist, score each option (1-5):

| Criteria      | Weight | Option A | Option B |
| ------------- | ------ | -------- | -------- |
| Testability   | 25%    |          |          |
| Readability   | 20%    |          |          |
| Performance   | 15%    |          |          |
| Consistency   | 15%    |          |          |
| Simplicity    | 15%    |          |          |
| Reversibility | 10%    |          |          |

**Choose highest weighted score, document decision in ADR (Architecture Decision Record)**

## Project Integration

### Learning the Codebase

1. **First Day**:

   - Run test suite successfully
   - Deploy to local environment
   - Read last 10 merged PRs
   - Identify code owners

2. **First Week**:

   - Fix one small bug
   - Add one test
   - Review 3 PRs
   - Document one unclear process

3. **First Month**:
   - Understand data flow
   - Know performance bottlenecks
   - Contribute meaningful feature
   - Mentor someone newer

### Communication Standards

- **PR/MR Description Template**:

  ```markdown
  ## What

  Brief description of changes

  ## Why

  Link to issue/requirement

  ## How

  Technical approach taken

  ## Testing

  How to verify changes

  ## Screenshots

  If UI changes

  ## Performance Impact

  Benchmarks if relevant

  ## Rollback Plan

  How to undo if issues arise
  ```

- **Commit Message Format**:

  ```
  type(scope): subject (max 50 chars)

  Body explaining why, not what (wrap at 72 chars)

  Fixes #123
  ```

  Types: feat, fix, docs, style, refactor, test, chore

### Code Review Guidelines

**As Reviewer**:

- Response within 4 hours during work hours
- Approve/Request changes within 24 hours
- Focus on correctness > style
- Suggest, don't demand
- Provide examples for complex feedback
- Check for security issues first

**As Author**:

- Self-review first
- Keep PRs under 400 lines
- Respond to feedback within 24 hours
- Don't take feedback personally
- Update based on feedback or explain why not

## Quality Gates

### Definition of Done

- [ ] Acceptance criteria met
- [ ] Tests written and passing (unit + integration)
- [ ] Code coverage maintained or improved
- [ ] Documentation updated (code + user-facing)
- [ ] Performance benchmarks pass
- [ ] Security scan pass
- [ ] Accessibility standards met (if UI)
- [ ] Feature flag configured (if applicable)
- [ ] Monitoring/alerts configured
- [ ] Reviewed by 2+ team members
- [ ] No unresolved TODOs without issue numbers
- [ ] Changelog updated

### Test Strategy

```python
# Test naming convention
def test_should_[expected]_when_[condition]():
    # Arrange
    # Act
    # Assert
```

- **Test Pyramid**:

  - 70% Unit tests (fast, isolated)
  - 20% Integration tests (API, database)
  - 10% E2E tests (critical paths only)

- **Test Guidelines**:
  - Test behavior, not implementation
  - One assertion per test when possible
  - Use builders/factories for test data
  - Tests should be deterministic
  - No conditional logic in tests
  - Mock external dependencies
  - Test edge cases and error paths

### Security Checklist

- [ ] Input validation on all user data
- [ ] SQL injection prevention (parameterized queries)
- [ ] XSS prevention (escape output)
- [ ] Authentication/authorization checks
- [ ] Sensitive data encrypted at rest
- [ ] Secrets in environment variables
- [ ] Dependencies scanned for vulnerabilities
- [ ] Rate limiting implemented
- [ ] CORS properly configured
- [ ] Security headers set

## Monitoring & Observability

### Required Telemetry

```javascript
// Structured logging example
logger.info("Operation completed", {
  operation: "user_signup",
  duration_ms: 145,
  user_id: user.id,
  trace_id: context.traceId,
  environment: process.env.NODE_ENV,
});
```

- **Logs**: Structured, with trace IDs
- **Metrics**: Business + technical KPIs
- **Traces**: Distributed tracing for requests
- **Events**: User actions and system events

### Alert Philosophy

- Alert on symptoms, not causes
- Every alert must be actionable
- Include runbook link in alert
- No more than 5 critical alerts per service
- Alert fatigue is a bug

## Incident Response

### When Things Break

1. **Acknowledge** - Claim incident in channel
2. **Assess** - Severity (P0-P4) and blast radius
3. **Communicate** - Status page + stakeholders
4. **Mitigate** - Stop bleeding, don't fix root cause yet
5. **Resolve** - Verify normal operations
6. **Learn** - Blameless postmortem within 48 hours

### Postmortem Template

```markdown
## Incident Summary

- Duration:
- Impact:
- Root Cause:

## Timeline

[Minute-by-minute during incident]

## What Went Well

[Things that helped]

## What Went Wrong

[Contributing factors]

## Action Items

[Specific improvements with owners and dates]

## Lessons Learned

[Knowledge to share with team]
```

## Continuous Improvement

### Weekly Team Rituals

- **Monday**: Planning and goal setting
- **Wednesday**: Tech debt review
- **Friday**: Learning hour (demos, articles, courses)

### Monthly Practices

- Dependency updates
- Performance review
- Security audit
- Documentation day
- Chaos engineering exercise

### Metrics to Track

- Lead time for changes
- Deployment frequency
- Mean time to recovery (MTTR)
- Change failure rate
- Code review turnaround
- Test coverage trend
- Technical debt ratio

## Important Reminders

### NEVER

- ❌ Use `--force` or `--no-verify` without team agreement
- ❌ Disable tests instead of fixing them
- ❌ Commit code that doesn't compile
- ❌ Store secrets in code
- ❌ Ignore security warnings
- ❌ Deploy on Friday after 3 PM
- ❌ Make assumptions - verify with data
- ❌ Optimize without profiling first
- ❌ Copy-paste without understanding
- ❌ Skip code review, even for "urgent" fixes

### ALWAYS

- ✅ Commit working code incrementally
- ✅ Write tests for bugs before fixing
- ✅ Update documentation as you go
- ✅ Learn from existing implementations
- ✅ Ask for help after 3 failed attempts
- ✅ Consider the next developer (might be you)
- ✅ Leave code better than you found it
- ✅ Celebrate wins, learn from failures
- ✅ Share knowledge proactively
- ✅ Take breaks and maintain work-life balance

## Resources

### Required Reading

- [The Pragmatic Programmer](https://pragprog.com/titles/tpp20/)
- [Clean Code](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
- [Site Reliability Engineering](https://sre.google/books/)
- [A Philosophy of Software Design](https://www.amazon.com/Philosophy-Software-Design-John-Ousterhout/dp/1732102201)

### Tools & References

- [12 Factor App](https://12factor.net/)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Semantic Versioning](https://semver.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Architecture Decision Records](https://adr.github.io/)

---

_"Make it work, make it right, make it fast" - Kent Beck_

_Last Updated: [08-08-2025]_
_Version: 1.0.0_
