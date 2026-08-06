Run a standalone security audit on $ARGUMENTS (or uncommitted changes if no argument).

The Security Auditor examines the change set for:
- Injection vulnerabilities (SQL, NoSQL, command, template)
- Authentication and authorization flaws
- Secret handling and leakage
- Unsafe deserialization
- SSRF and network boundary issues
- Path traversal
- Dependency risk
- Insecure defaults
- Sensitive data exposure

Findings are ranked Critical/High/Medium/Low with file:line locations and concrete remediations. If the audit finds nothing, it will say so plainly.

Omit arguments to audit uncommitted changes. Provide a file or directory path to scope the audit.
