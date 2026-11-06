# Linux from Basics to Professional

A practical Linux command reference written from a Linux Administrator I → Linux Administrator II perspective, with a separate specialization section for authorized security testing.

## Main reference

[Open the Linux Administration Reference](./Linux-Administration-Reference.md)

## Coverage

The reference covers:

- Navigation, file operations, search, copy, synchronization, and archives.
- Text processing for logs and configuration files.
- Users, groups, ownership, and permissions.
- Process inspection, signals, and resource triage.
- `systemd` services, unit discovery, and journal logs.
- Network addresses, routes, sockets, connectivity, and DNS.
- Disks, filesystems, mounts, and storage troubleshooting.
- Debian/Ubuntu package management.
- SSH, sudo, and firewall fundamentals.
- Fast troubleshooting workflows.
- Security-testing specializations including web, mobile, cloud, containers, wireless, internal network, reverse engineering, and reporting.

## How to use it

The document is organized by task. For each command, it aims to explain:

- What the command does.
- When it is useful.
- A practical example.
- Operational context where relevant.

Treat examples as templates. Check command syntax, options, and system-specific behavior against the local manual pages and distribution documentation.

## Safety

Some examples perform network reconnaissance, access-control changes, file operations, or security testing. Confirm authorization and operational scope before running them. Never copy sample credentials or target values into a production environment without review.

## Project format

This repository is intentionally document-first. The primary deliverable is a Markdown reference; there is no application build or runtime dependency.

## License

The original written reference is licensed under **Creative Commons Attribution 4.0 International (CC BY 4.0)**. See [LICENSE](./LICENSE).

You may share and adapt the licensed material, including commercially, provided you give appropriate attribution, link to the license, and indicate changes. The license does not cover third-party trademarks or material that the repository owner does not have authority to license.
