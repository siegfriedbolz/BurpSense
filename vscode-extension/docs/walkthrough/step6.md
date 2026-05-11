# Browse and Filter Issues

**Issues View Structure:**

BurpSense organizes issues in a three-level hierarchy for easy browsing:

1. **Severity Level** (HIGH, MEDIUM, LOW, INFORMATION)
2. **Issue Name** (with instance count, e.g. "SQL injection (3 instances)")
3. **Individual URLs** (each affected endpoint with short ID)

**Example:**
```
📁 HIGH
  ⚠️ SQL injection (3 instances)
    🔗 https://api.example.com/login [a9b2921d]
    🔗 https://api.example.com/search [f4e8c2a1]
    🔗 https://api.example.com/admin [b3c4d5e6]
  ⚠️ Cross-site scripting (reflected) (1 instance)
    🔗 https://example.com/search [c7d8e9f0]
```

**Navigation:**
- Click **severity folders** to expand/collapse issue categories
- Click **issue names** to see all affected URLs
- Click **individual URLs** to view full advisory details
- Expansion state is preserved when refreshing

**Search Issues:**
1. Click the **search icon** (🔍) in the Issues View toolbar
2. Type to search by name, URL, severity, or issue ID
3. Results update instantly
4. Active filters show in the right status bar
5. Click the **X icon** to clear the search

**Filter Presets:**
1. Click the **filter preset icon** (≣) in the toolbar
2. Choose from quick presets:
   - **All Issues** - Shows everything
   - **High Severity Only** - Critical findings
   - **High + Medium** - Important issues
   - **Recommended** - High/Medium, in-scope, high confidence

**Manual Filters:**
- **Severity filter** (⚠️) - Set minimum severity level
- **Confidence filter** (✓) - Set minimum confidence level  
- **Scope toggle** (🎯) - Show only in-scope issues

**Refresh:**
Click the **refresh icon** (↻) to fetch latest issues from Burp Suite.

**Benefits of Grouped View:**
- Quickly see which issues are most widespread
- Collapse duplicate issues to reduce clutter
- Focus on unique security problems
- Easily compare multiple instances of same issue

---

**About this build:** **BurpSense for Cursor** is maintained by **Siegfried-Thor Bolz** as a fork of **Arqsz**’s original **BurpSense** ([TheArqsz/BurpSense](https://github.com/TheArqsz/BurpSense)). Reference compatibility: **Cursor 3.3.30** and **Burp Suite v2026.4.2**.