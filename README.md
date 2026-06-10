# Contribution #1: PreventionWs SOAP service bypasses the `_prevention` read privilege check

**Contribution Number:** 1
**Student:** Dinesh Reddy
**Issue:** [carlos-emr/carlos #2761](https://github.com/carlos-emr/carlos/issues/2761)
**Status:** Phase I Complete

---

## Why I Chose This Issue

I chose issue #2761 in the OSCAR EMR project because it is a clearly scoped security bug labeled `good first issue`, `help wanted`, and `security`. The problem is specific and easy to state: four methods in `PreventionWs.java` (`getPrevention`, `getUpdatedAfterDate`, `getByDemographicIdUpdatedAfterDate`, and `getPreventionsByProgramProviderDemographicDate`) call `getLoggedInInfo()` but never verify that the authenticated consumer actually has read access to prevention/immunization data. As a result, any authenticated user — including one with a revoked or compromised OAuth token — can retrieve sensitive patient health information (PHI). "Fixed" means each of these methods performs a `_prevention` read privilege check and rejects unauthorized callers before returning any data.

This issue matches what I can realistically learn and ship in this cycle. The codebase is Java/SOAP, which I'm newer to, but the change is small and pattern-based rather than requiring deep system knowledge — and the issue points to #2471, the identical flaw already fixed in `AllergyWs`, which gives me a working reference for the exact privilege-check pattern to apply. I want to learn how authorization is enforced in a real production healthcare system, and adding a `securityInfoManager.hasPrivilege(...)` gate is a focused way to see how privilege checks guard sensitive endpoints.

---

## Understanding the Issue

### Problem Description

Four methods in `PreventionWs.java` return prevention/immunization records without first checking that the caller has the `_prevention` read privilege, so authenticated-but-unauthorized users can access patient PHI.

### Expected Behavior

Each affected method should confirm the logged-in consumer has `_prevention` read access before returning data, and reject the request (e.g. by throwing a security exception) when that privilege is missing.

### Current Behavior

The methods call `getLoggedInInfo()` and delegate straight to the manager without any privilege gate, so any authenticated user receives the data regardless of their permissions.

### Affected Components

- `PreventionWs.java` — the four methods: `getPrevention`, `getUpdatedAfterDate`, `getByDemographicIdUpdatedAfterDate`, `getPreventionsByProgramProviderDemographicDate`
- `PreventionManagerImpl` (two methods here already perform the check correctly — a reference)
- `securityInfoManager` / privilege-checking infrastructure
- Reference fix: `AllergyWs` (issue #2471)

---

## Reproduction Process

### Environment Setup

_(To be completed in Phase II — local OSCAR build/setup notes will go here.)_

### Steps to Reproduce

_(To be completed in Phase II.)_

### Reproduction Evidence

- **Commit showing reproduction:** _(Phase II)_
- **Screenshots/logs:** _(Phase II)_
- **My findings:** _(Phase II)_

---

## Solution Approach

### Analysis

_(To be completed in Phase II.)_

### Proposed Solution

The likely fix is to add a privilege check at the start of each affected method, mirroring the already-fixed `AllergyWs` (#2471) and the two correct methods in `PreventionManagerImpl`:

```java
if (!securityInfoManager.hasPrivilege(loggedInInfo, "_prevention", "r", null)) {
    throw new SecurityException("missing required sec object");
}
```

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** Four `PreventionWs` methods expose PHI without verifying the caller's `_prevention` read privilege.

**Match:** `AllergyWs` (#2471) and two methods in `PreventionManagerImpl` already implement the correct `hasPrivilege` pattern.

**Plan:**
1. Add the `_prevention` read privilege check to `getPrevention`.
2. Add the same check to `getUpdatedAfterDate`, `getByDemographicIdUpdatedAfterDate`, and `getPreventionsByProgramProviderDemographicDate`.
3. Add/adjust tests covering authorized vs. unauthorized access.

**Implement:** _(Links to branch/commits in Phase III.)_

**Review:** _(Self-review against the project's CONTRIBUTING guidelines in Phase III.)_

**Evaluate:** _(Verification approach in Phase III.)_

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: Authorized user (has `_prevention` read) receives data
- [ ] Test case 2: Unauthorized user is rejected with a security exception
- [ ] Test case 3: _(TBD in Phase III)_

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

_(To be completed in Phase III.)_

---

## Implementation Notes

### Week [X] Progress

_(To be completed as work begins.)_

### Code Changes

- **Files modified:** _(TBD)_
- **Key commits:** _(TBD)_
- **Approach decisions:** _(TBD)_

---

## Pull Request

**PR Link:** _(TBD in Phase IV)_

**PR Description:** _(TBD in Phase IV)_

**Maintainer Feedback:**
- _(Date)_: _(Summary)_

**Status:** _(Not yet submitted)_

---

## Learnings & Reflections

### Technical Skills Gained

_(To be completed.)_

### Challenges Overcome

_(To be completed.)_

### What I'd Do Differently Next Time

_(To be completed.)_

---

## Resources Used

- [Issue #2761](https://github.com/carlos-emr/carlos/issues/2761)
- [Issue #2471 — the reference fix in AllergyWs](https://github.com/carlos-emr/carlos/issues/2471)
