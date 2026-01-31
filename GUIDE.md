# Guide: Adding Questions to PerfectPrep

This guide explains how to format questions and answers in the README.md file.

---

## Basic Format

```markdown
### Q: What is Docker?

Docker is a platform for developing, shipping, and running applications in containers.
Containers package code and dependencies together, ensuring consistent behavior across environments.
```

---

## Using Collapsible Sections

For longer answers, use collapsible sections:

```markdown
### Q: Explain the Linux boot process

<details>
<summary>Answer</summary>

1. BIOS/UEFI loads
2. Bootloader (GRUB) starts
3. Kernel initializes
4. Init system (systemd) starts
5. Services and targets load

</details>
```

---

## Using Tables

For comparison questions:

```markdown
### Q: Docker vs Virtual Machines?

| Aspect | Docker | Virtual Machine |
|--------|--------|-----------------|
| Size | Lightweight (MBs) | Heavy (GBs) |
| Boot Time | Seconds | Minutes |
| Isolation | Process-level | Full OS |
```

---

## Using Code Blocks

For commands or code snippets:

```markdown
### Q: How do you check disk usage?

Use the `df` command:

\`\`\`bash
df -h
\`\`\`

Or for directory-specific usage:

\`\`\`bash
du -sh /var/log
\`\`\`
```

---

## Tips

1. Keep answers concise but complete
2. Use bullet points for multi-step answers
3. Include practical examples where possible
4. Add code blocks for commands
5. Use tables for comparisons
