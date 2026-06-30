# Contribution [#2979]: [Return NONE instead of null in DisplayPersonalInfoAppointment2Action.toggle()]

**Contribution Number:** [1]  
**Student:** [Lizaveta Khalipava]  
**Issue:** [[GitHub issue link](https://github.com/carlos-emr/carlos/issues/2650)]  
**Status:** [Phase IV] [Complete]

---

## Why I Chose This Issue

I chose issue #2650 "DisplayPersonalInfoAppointment2Action.toggle() returns null instead of NONE" because it offers a great opportunity to get hands-on experience with the Struts 7 framework and action result management within the CARLOS codebase. As I am currently deepening my knowledge of Java-based web application infrastructure, this issue provides a clear, contained task that directly impacts response reliability.

I’m interested in this because:
1. I want to better understand how Struts 7 handles direct response paths and why Action.NONE is preferred over returning null to avoid undefined result resolution.

1. The issue provides precise instructions and points to the exact file (DisplayPersonalInfoAppointment2Action.java) and line (81) needing adjustment, making it ideal for a first contribution.

1. The maintainer has explicitly linked this to similar patterns fixed in other parts of the project (#2574 and #2594), signaling that this is part of a standardized effort to stabilize the codebase.

From reading the issue thread, I understand that the current implementation returns null after writing to the response, which can lead to unpredictable behavior in Struts. My plan is to update the toggle() method to return NONE and verify that the struts-scheduling.xml mapping is configured correctly.

I have reviewed the linked companion issues to ensure my fix aligns with the project’s established patterns. I am ready to begin working on this and will ensure the changes follow the architectural guidelines discussed in the thread.

---

## Understanding the Issue

### Problem Description

The toggle() method in DisplayPersonalInfoAppointment2Action writes directly to the HTTP response but returns a bare null value instead of returning the proper Struts constant NONE. Because Struts 7 handles action result mapping strictly, returning null leaves the result resolution undefined. This can cause the framework to accidentally try to find a default result mapping or append unwanted HTML error content onto a response that has already been successfully written.

### Expected Behavior

After an action successfully writes data directly to the response stream, it should return Action.NONE. This explicitly instructs Struts 7 that the response is fully handled and that no further result processing, view rendering, or content appending should take place.

### Current Behavior

The method returns null. This triggers unpredictable result resolution in Struts 7, creating a risk that Struts will attempt to resolve the null mapping and corrupt the client response by trailing it with error page fragments or layout HTML.

### Affected Components

src/main/java/io/github/carlos_emr/carlos/provider/web/DisplayPersonalInfoAppointment2Action.java (Specifically the toggle() and execute() methods).

src/main/resources/struts-scheduling.xml (The configuration file defining result mappings for this action area).

---

## Reproduction Process

### Environment Setup

Prerequisites : 
- Docker Desktop installed and running
- VS Code with the Dev Containers extension
- Git
- Ports 8080 and 3306 must be available

### Steps to Reproduce

1. Navigate to the appointment scheduling interface or trigger the toggle feature associated with personal info display settings.

2. Intercept or monitor the server network traffic/logs when DisplayPersonalInfoAppointment2Action executes.

3. Observed result: While the initial response data is written, the console logs or raw network responses show Struts 7 executing additional result resolution workflows for a null action string, risking appended markup overhead or warning flags in the application log.

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork](https://github.com/carlos-emr/carlos/commit/56ff5c2d6ef97f28baa42116278c3a897ef7afdf)

---

## Solution Approach

### Analysis

The root cause is a deviation from Struts 7 best practices regarding direct-response actions. When a method bypasses the standard view layer (JSP/FreeMarker) to stream raw data directly to the client, Struts needs an explicit signal to halt execution. Returning null is a legacy pattern that Struts 7 treats as undefined.

### Proposed Solution

1. Update Java Action Source: Modify DisplayPersonalInfoAppointment2Action.java line 81 to explicitly return the Struts framework constant NONE instead of null.

2. Verify Configuration: Inspect struts-scheduling.xml to confirm that the action's result mappings either completely omit a mapping for "none" (defaulting to a no-op as required) or handle it explicitly as an empty forward to prevent additional framework interceptors from writing to the output stream.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** The DisplayPersonalInfoAppointment2Action.toggle() method writes directly to the response but returns a bare null. In Struts 7, returning null leaves result resolution undefined, which causes the framework to look for a default view mapping and risks appending accidental HTML error content or layout fragments to a response that has already been sent to the client.

**Match:** This matches an identical pattern addressed across similar direct-response paths within the CARLOS codebase, specifically:

#2574: Fixed null-return patterns in lab direct-response actions.

#2594: Fixed the exact same issue in EChartPrint2Action.

Guidelines in CLAUDE.md state that a bare null return string is unreliable following direct response generation in Struts 7.

**Plan:** [Step-by-step implementation plan]
1. Modify Target Action: Update DisplayPersonalInfoAppointment2Action.java to change the return statement of toggle() from return null; to return NONE; (or ActionSupport.NONE).

2. Review Configuration: Verify that src/main/resources/struts-scheduling.xml correctly maps this action without defining a forward for "none", maintaining a clean no-op termination.

3. Run Regression suite: Execute the newly introduced unit tests to guarantee that both the privilege gate checks and the updated NONE return logic work properly.

**Implement:** [[Link to your commit](https://github.com/carlos-emr/carlos/pull/2979/changes/9c61a4315a7398c215aca2f5a201c48aaff7b349)] [[Link to your commit](https://github.com/carlos-emr/carlos/pull/2979/changes/cee1a2179658a03d22e58cce4ba467c5e773dd92)]

**Review:** 

[ ] Core logic updates follow strict Java code standards specified by CARLOS.

[ ] Code changes avoid introducing hardcoded string literals where existing constants (ActionSupport.NONE) are available.

[ ] PR links accurately back to issue #2650.

[ ] DCO sign-off followed.

**Evaluate:** 

Compilation and verification will be validated by executing the unit test class DisplayPersonalInfoAppointment2ActionUnitTest. Green tests ensure the code change meets expected execution behavior and matches framework contracts.

---

## Testing Strategy

### Unit Tests

[x] Test Case 1: shouldReturnNoneAndSetTrue_whenAttributeAbsent
Verifies that a first-time execution initializes showPersonal to true in the session and safely returns ActionSupport.NONE.

[x] Test Case 2: shouldReturnNoneAndSetFalse_whenPreviouslyTrue
Ensures that when the user profile state is already set to true, a call to toggle() properly flips it to false and still terminates with ActionSupport.NONE.

[x] Test Case 3: shouldReturnNoneAndSetTrue_whenPreviouslyFalse
Confirms state cycling works properly by flipping an existing false session attribute back to true while continuing to return ActionSupport.NONE.

[x] Test Case 4: shouldThrowSecurityException_whenPrivilegeMissing
Pins the security constraint residing in execute(), confirming a SecurityException is thrown and aborts processing if the caller lacks _demographic read rights.

### Integration Tests

[ ] Integration Scenario 1: Deploy the application changes to a container, log into a provider account, and fire the showPersonal toggle action. Confirm via network tools that the HTTP response headers return a clean status code and zero trailing content body bytes.

[ ] Integration Scenario 2: Attempt to run the endpoint under an account lacking _demographic view roles to ensure the framework catches and properly rejects the transaction upstream.

### Manual Testing

Ran mvn clean test -Dtest=DisplayPersonalInfoAppointment2ActionUnitTest against the local sandbox environment.

Checked the console log outputs to confirm that Struts 7 mapping errors disappear and the mock context runs successfully.

---

## Implementation Notes

### Week [1] Progress

**What was built/done:**  Dedicated this week to issue selection and local environment alignment. Explored the repository to locate an approachable bug, settling on issue #2650 due to its clear scope.

**Challenges faced:**  Setting up a legacy/large-scale enterprise Java application locally can surface configuration drift. Ensuring the correct JDK version and Maven wrapper settings were aligned took some troubleshooting.

**Decisions made:**  Relied closely on the main Readme and the container one for repository guidelines and how to set up the program.

### Week [2] Progress

**What was built/done:**  Focused on tracing the execution path of DisplayPersonalInfoAppointment2Action.java to reproduce the logic flaw.

**Challenges faced:**  Struts 7 handles undefined null returns behind the scenes, catching the exact edge-case side effect required deep tracing of the result resolution chain.

**Decisions made:**  Analyzed how companion issues #2574 and #2594 solved the same problem. Decided that returning the explicit framework constant ActionSupport.NONE was the safest architectural approach.

### Week [3] Progress

**What was built/done:**  Implemented the code fix changing the return type to NONE. Drafted the initial unit test suite to pin down the session toggle states.

**Challenges faced:**  Writing comprehensive unit tests for the first time. Mocking static contexts required learning how to handle thread-local web contexts in isolated test environments.

**Decisions made:**  Structured the tests using a custom withMockedContext consumer pattern to clean up static mocks automatically after each test execution block.

### Week [4] Progress

**What was built/done:**  Packaged the changes into a clean branch, opened Pull Request #2979, and rapidly iterated through the maintainer's code review feedback.

**Challenges faced:**  Encountered pipeline verification failures due to missing Developer Certificate of Origin (DCO) commit sign-offs. Resolving this required learning how to selectively upstream fixes, rewrite git histories, and utilize git cherry-pick.

**Decisions made:**  Refactored the test suite to invoke toggle() directly for state validation while preserving execute() strictly for testing the privilege intercept gate, fulfilling the reviewer's exact preferences.

### Code Changes

- **Files modified:** 
1. src/main/java/io/github/carlos_emr/carlos/provider/web/DisplayPersonalInfoAppointment2Action.java
2. src/main/webapp/WEB-INF/classes/struts-provider.xml
3. src/test/java/io/github/carlos_emr/carlos/provider/web/DisplayPersonalInfoAppointment2ActionUnitTest.java
- **Key commits:** [Link to important commits]https://github.com/carlos-emr/carlos/pull/2979/commits

---

## Pull Request

**PR Link:** [[GitHub PR URL when submitted](https://github.com/carlos-emr/carlos/pull/2979)]

**PR Description:** Fixes the return null to return NONE in DisplayPersonalInfoAppointment2Action.toggle().

**Maintainer Feedback:**
- [6-24-2026]: [Summary of feedback received]
    1. Include struts configuration
    2. Change the tests to call the toggle() function instead of execute()
    3. PR title to match the issue name
    4. Include more docstrings explaining code purpose and functionality
- [6-24-2026]: [How you addressed it]

**Status:** [Merged]

---

## Learnings & Reflections

### Technical Skills Gained

1. Creating my first PR, understanding the process, and iterating through feedback and clean solution.
2. Navigating an extensive codebase and learning about proper code organization. 

### Challenges Overcome

1. Each commit had to be signed off, which failed the tests when I created a PR. In between that I also had to merge from upstream, which made my sign offs messy at first, and had to utilize a new skill of cherry picking. 
2. Writing unit tests for the first time and understanding how to trigger certain behavior.

### What I'd Do Differently Next Time

Read very carefully through the Readme files to familiarize with the codebase and styles maintainers want to have and not be afraid to ask them questions. 

---