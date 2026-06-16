# Contribution 1: allow downloading insecure, (Fetch via HTTP if HTTPS is unavailable)

Note: Contributing with main developer account nathant27

**Contribution Number:** 1
**Student:** Nathan Tam
**Issue:** [[GitHub issue link]](https://github.com/kubernetes/minikube/issues/6692#event-26436031703) 
**Status:** Phase 1 Complete

---

## Why I Chose This Issue

I chose this issue because I wanted to work more on a more substantial project in golang, being my recent language of choice for backend development in my own time.
I'm seeking to develo my golang skills in larger codebases and following good maintainable practices, and I hope to contribute to this project in a meaningful way by the time I'm done.

---

## Understanding the Issue

### Problem Description

When downloading kubectl and other kube tools, fails to download on certain LAN infrastructure configuration. 

### Expected Behavior

Would ideally like a force option to download insecurely

### Current Behavior

Gives an error that it failed to download

### Affected Components

the download module, in pkg/minikube/download/iso.go

---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. make folder repro-issue and file main.go with code:
```
package main

import (
	"fmt"
	"net/http"
	"net/http/httptest"
	"os"

	"k8s.io/minikube/pkg/minikube/download"
)

func main() {
	// httptest.NewTLSServer serves over HTTPS using an in-memory, self-signed
	// certificate. To the default Go http client this looks identical to a
	// corporate proxy re-signing traffic with a CA the machine doesn't trust.
	srv := httptest.NewTLSServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// We never get here: the TLS handshake fails first. Present anyway so
		// that, once the insecure fix lands, the download can actually succeed.
		fmt.Fprintln(os.Stderr, "server: received request for", r.URL.Path)
		_, _ = w.Write([]byte("not-a-real-binary"))
	}))
	defer srv.Close()

	fmt.Println("Untrusted HTTPS mirror listening at:", srv.URL)
	fmt.Println("Attempting to download kubectl v1.17.2 through it...")

	// srv.URL is the binaryURL ("--binary-mirror"). download.Binary builds the
	// real release path + checksum URL under it and fetches via go-getter.
	_, err := download.Binary("kubectl", "v1.17.2", "linux", "amd64", srv.URL)
	if err != nil {
		fmt.Println("\nReproduced the issue. Download failed with:")
		fmt.Println(err)
		os.Exit(1)
	}

	fmt.Println("\nDownload succeeded -- issue NOT reproduced (insecure path is allowed).")
}
```

2. run this script with go run repro-issue
3. When trying to download from mock TLS server, fails to downlaod because certified signed by unknown entity.

### Reproduction Evidence

- **Commit showing reproduction:** Running script above. Not yet commited.
- **Screenshots/logs:**
- Logs include "  ... Get "https://127.0.0.1:.../kubectl.sha256": tls: failed to verify certificate:
  x509: certificate signed by unknown authority"
- **My findings:** Replication fairly easy by using a mock TLS server in memory which self signs. When downloading from this, causes the same issue as stated.

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
