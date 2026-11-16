[한국어](../09_step4-response.md) | **English**

# ④ Visualize · Responsiveness

For each request-response relationship defined on tab ③, shows the time from request to response over time, plus missing, duplicate and orphan responses.

**How to use**

1. Relationships are created with "Add Relationship" in ③ Test Analysis Settings; here you only tick which ones to show.
2. X is the request time, Y the response time (ms).
3. Click a point or anomaly marker; "Selected Event Details" at the bottom right shows the request/response frames and why it was judged so.
4. "View in Timeline" in the details jumps straight to that frame.
5. "Export Selected Data" saves the shown relationships' events as CSV/Excel.

**Notes**

- No response before the next request is missing (red X), two or more is duplicate (orange circles), and one belonging to no request is orphan (purple triangle).
- Missing, duplicate and orphan events are never drawn as 0, and points around a gap are not joined.
- Time bookmarks and ranges are shared by all four visualization tabs; right-click the graph to add, edit or delete them.
- A relationship whose messages cannot be found is marked "Unavailable" and not analyzed.

---

[← ④ Visualize · Continuity](08_step4-continuity.md) · [Contents](../../README.en.md#user-guide) · [⑤ Analysis Results · Report →](10_step5.md)
