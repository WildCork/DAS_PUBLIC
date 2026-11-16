[한국어](../01_workflow.md) | **English**

# Overall Workflow

For a single packet to show up on screen as a meaningful message, four things have to line up together: devices/sockets, links, IDL, and the PCAPNG itself.

**How to use**

1. The sending/receiving IP:Port is read from the Ethernet/IP/UDP-TCP headers.
2. It's matched against the same IP:Port pair registered in ② Link Setup.
3. The message ID and fields are decoded using that link's assigned IDL.
4. The decoded result is then reused across Timeline, Periodicity, Continuity, and the report.

**Notes**

- A PCAPNG opening without error is not the same thing as the interpretation being correct -- always also check that devices/links/IDL actually match the real capture.

---

[← Quick Start](00_quick-start.md) · [Contents](../../README.en.md#user-guide) · [Step 0: Test Config File Setup →](02_config-files.md)
