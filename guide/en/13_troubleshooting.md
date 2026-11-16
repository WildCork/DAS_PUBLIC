[한국어](../13_troubleshooting.md) | **English**

# Troubleshooting

A collection of common symptoms and where to check -- most are configuration issues in the earlier steps (①~③).

**Notes**

- Loaded Lua but no IDL appears → check that a matching <name>.idl for each protos name is in the same folder as the Lua file.
- PCAPNG loaded but the message tree is empty → check that each link's IP:Port matches the actual packets, and that an IDL is assigned to each link.
- Message ID or detail decoding fails → compare the IDL's endianness/PACK_SIZE/header structure against the actual sending struct.
- Periodicity/Continuity tabs are empty → check the IDL's PERIODIC list, the actual occurrence count (2 or more), and the step ③ filter state.
- The smart filter hides too much → turn it off or raise the threshold percentage.
- Report shows "No such occurrence" → check that the specified Nth occurrence isn't larger than the actual occurrence count.
- Opened the config JSON on another PC and it can't find the IDL → the JSON only stores the IDL path, so move the IDL files along with it.
- A button is grayed out and won't click → most such buttons need something "selected/checked first". Edit Device / Delete Device / Remove IDL / Delete Selected-style buttons only enable once a row is selected (or checked) in the matching list, and smart-filter buttons are always disabled while a live analysis session is in progress.
- Devices/IDLs/links registered earlier disappeared after Load Config → this is expected, not a malfunction. Loading a config (JSON or Lua) always replaces devices/IDLs/links wholesale rather than appending to what's already there — if you want to keep the existing setup, save it as a config first before loading a new one.
- ① shows devices/IDLs, but ② Link list is empty → either endpoints.lua's protos entry didn't have 2 or more valid socket keys (that whole protocol was skipped), or the config JSON's links array is empty — add links directly in ②.
- Messages show but no IDL fields appear (only raw bytes) → the link has no IDL assigned ("(None)"), or it couldn't auto-find a matching .idl by protos name — assign the IDL combo for that link yourself in ②.
- The smart filter buttons (toggle and threshold settings alike) won't click → always disabled while a live capture is in progress. Reopen the final PCAPNG in Result mode after finishing capture to use them again.
- Double-clicking a message row in the IDL tree does not open the "Edit IDL Message" window → you clicked an IDL file row or an already-configured rule field row (level 3) — try again on a level-2 (Message ID) row. It opens for every message regardless of periodicity (for a non-periodic message only the "Smart Filter Targets" tab in that window is disabled), and the IDL tree right-click menu no longer exists.
- Applied/saved a message identifier rule but the ③/④ step screens look unchanged → that's expected. The rule takes effect immediately only in the IDL tree and ".das-idl.json" -- already-loaded PCAPNG analysis results only change once you reload the PCAPNG in ③ Test Analysis Settings; watch for the "reload needed" notice on the ①/③ tabs.
- Identifier combinations exceed 10 so the rule isn't applied on a specific connection → after reloading the PCAPNG, that connection's message row shows "Over 10 identifier combinations — rule not applied". You likely picked a field that changes on nearly every occurrence (a free-running counter, timestamp-like field) -- remove it in Devices/IDL Setup and keep only fields that repeat across a handful of values (mode/status flags) -- another connection sharing the same IDL stays normally applied if its own count is 10 or under.
- A warning icon shows in the IDL tree → the same-folder ".das-idl.json" auxiliary file has a problem (JSON syntax error, unsupported version, missing Message ID, struct-name mismatch, missing field path, ...) -- check the row's tooltip for the specific cause. The file is never auto-fixed or deleted.
- A reference line added by right-click doesn't show or can't be deleted → reference lines are only drawn and interactive in Line Chart mode (they don't appear in Box Plot mode). To delete one, right-click exactly on its icon and choose "Delete Reference Line" — right-clicking empty space just adds another one.
- Loading a report format fails with an error → the format is re-linked to the current project by device name, IDL name, and message ID, and if even one can't be found the whole load fails rather than skipping just that item — first match ①/② to the same device/IDL names the format was made with.
- The interface to use for live analysis is missing from the list, or an error appears → network adapter enumeration itself failed — check the red error text on screen, and try running as administrator or checking the network adapter's status.
- An error mentioning wpcap.dll/Packet.dll (Npcap) appears → live analysis requires an Npcap (or WinPcap-compatible) driver to be installed. Install Npcap and run again — Result mode, which opens a saved PCAPNG, works fine without this driver.
- A PING test shows failure → this only means there was no ICMP reply, not that the device can't send UDP/TCP data (it may simply block ICMP). Check whether traffic is actually flowing separately by starting a capture and watching for received packets.
- A message is missing from the Timeline only (the step ③ filter and Smart Filter are both on) → right-click the column header of the device that sent it, open "Manage Sent Messages Shown", and check whether it has been unchecked — this setting applies to the Timeline only, so the Periodicity and Continuity tabs were never affected in the first place.
- The visibility you set in "Manage Sent Messages Shown" is gone the next time you open it → that is expected. This setting is temporary state that is never written to a file, so it resets to "show all" every time you load a new PCAPNG or start a new live session.

---

[← Mouse and Hidden Operations](12_mouse-hidden.md) · [Contents](../../README.en.md#user-guide) · [Shortcuts and Supported File Types →](14_shortcuts.md)
