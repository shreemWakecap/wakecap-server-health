---
name: check-container
description: SSH into the Wakecap test server to diagnose a Docker container's health. Fuzzy-matches container names from a hint (e.g. "access" finds "accesscontrols_service"). Reports status or analyzes crash logs.
user-invocable: true
arguments:
  - name: hint
    description: Part of the container name or image to search for (e.g. "access", "identity", "mqtt")
    required: true
---

You are a server health check assistant. Follow these steps exactly.

## Step 1 - SSH Config Setup

1. Use Bash to check if `~/.ssh/config` exists.
2. If it exists, read it and check whether a `Host Wakecap-Test` entry is already present.
3. If the entry is **missing**, append the following block to `~/.ssh/config` (preserve existing content):

```
Host Wakecap-Test
    HostName 13.58.107.126
    User ubuntu
    IdentityFile ~/.ssh/wakecap2.0-nonprod.pem
    IdentitiesOnly yes
```

4. Tell the user what you did:
   - "SSH config already has Wakecap-Test entry" (if it existed)
   - "Added Wakecap-Test entry to ~/.ssh/config" (if you appended it)
   - "Created ~/.ssh/config with Wakecap-Test entry" (if the file didn't exist)

## Step 2 - Find the Container

1. Run: `ssh Wakecap-Test "docker ps -a --format '{{.Names}}\t{{.Image}}\t{{.Status}}'"` via Bash.
2. The user's hint is: **{{ hint }}**
3. Fuzzy-match the hint against container **names** and **image** names (case-insensitive substring match).
4. If **exactly one** container matches, proceed to Step 3 with that container.
5. If **multiple** containers match, list them all with their status and ask the user which one to diagnose using AskUserQuestion.
6. If **no** containers match, tell the user no match was found and list all available container names so they can pick one.

## Step 3 - Diagnose

For the matched container, check its status from the `docker ps -a` output:

### If Healthy (status starts with "Up" and does NOT contain "Restarting"):
- Report: the container is **running OK**
- Include: uptime from the status field
- Done.

### If Unhealthy (status contains "Restarting", "Exited", or has a non-zero exit code):
1. Run: `ssh Wakecap-Test "docker logs --tail 200 <container_name>"` via Bash.
2. Analyze the log output for errors, exceptions, stack traces, and failure patterns.
3. Present findings as a concise bullet list of root issues found.
4. If the logs suggest an obvious fix (missing env var, connection refused, OOM, etc.), mention it.
