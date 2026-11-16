[한국어](../12_mouse-hidden.md) | **English**

# Mouse and Hidden Operations

Various screens have actions reachable only by double-click, right-click, or drag, with no visible button — each was verified against the actual implementation below. Anything not listed here (e.g. right-clicking outside an axis title, drag-reorder in trees other than the legend) does not exist.

**Notes**

- [① Devices/IDL] Double-clicking a device row in the device list opens the device edit dialog; double-clicking a socket (child) row under it opens a separate socket-edit dialog for its IP/port.
- [② Link Setup] Double-clicking a row in the defined links list opens an edit dialog for that link's source/destination/IDL; clicking OK updates that same row in place rather than adding a new one.
- [① Devices / IDL] Double-clicking a message (Message ID) row in the IDL tree opens the "Edit IDL Message" window (one window with three tabs: Message Info, Additional Identifiers, and Smart Filter Targets) — it does not open on IDL file rows, on already-configured rule field rows (level 3), or on empty space. The IDL tree has no right-click menu.
- [④ Periodicity] Right-clicking the legend tree on the right shows one menu item that hides everything except whatever you clicked (a device, message kind, or variant).
- [④ Periodicity/Continuity] Clicking a point (node) on the graph doesn't immediately show a popup — a menu appears first, and you have to choose "View info about this node" (one occurrence's detail) or "View info about this periodic message" (that kind's overall statistics) etc. for the callout to be pinned.
- [④ Periodicity/Continuity] Right-clicking an empty area (inside the first quadrant) of the graph adds a new reference line (y=a) at that y value. Left-click-dragging an existing reference line's icon moves its value, and right-clicking that icon shows a "Delete Reference Line" menu item — reference lines are neither visible nor interactive in Box Plot mode.
- [④ Periodicity/Continuity] The X/Y axis title text itself is a button — clicking "Elapsed Time" or "Period"/"Frequency" opens a dialog for setting the axis range and grid spacing.
- [④ Timeline] Dragging the minimap on the left immediately scrolls the center list to that time position — the minimap itself never scrolls and always shows the whole span.
- [④ Timeline] Dragging a device column header left/right reorders that column, and the minimap's device order changes along with it — clicking without dragging just toggles that device on/off instead of reordering.
- [④ Timeline] Right-clicking a device column header brings up a single "<device> Manage Sent Messages Shown" menu item; choosing it opens a window where you pick, per link and identifier variant, which of that device's message kinds to hide — on the Timeline only. This setting is not saved and resets every time you load a new PCAPNG or live session.
- [⑤ Analysis Results > Report] Double-clicking a row in the timeline list or the check-item table opens its respective edit dialog.
- [⑤ Analysis Results > Report] Press and drag the sequence-number cell of the check-item table to reorder that item — pressing and dragging another cell doesn't reorder anything.
- [⑤ Analysis Results > Report] Selecting several adjacent rows in the check-item table and right-clicking shows whichever menu fits the selection's state: group into check stage/category (not yet grouped), rename (already all sharing one label), or ungroup (all already grouped).
- F1 always jumps to the help topic matching whatever step you're currently on (down to which Timeline/Periodicity/Continuity sub-tab is open, for ④ Visualize) — even if the help window is already open, pressing it again still navigates it to the new topic. Other shortcuts are collected in "Shortcuts and Supported File Formats".

---

[← ⑤ Analysis Results · Dashboard](11_step5-composite.md) · [Contents](../../README.en.md#user-guide) · [Troubleshooting →](13_troubleshooting.md)
