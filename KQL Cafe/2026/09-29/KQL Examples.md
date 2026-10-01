# Supplemental KQL Examples

This presentation uses intentionally simplified KQL examples to illustrate specific failure modes and reasoning patterns.

The queries below are fuller, operational examples from the [DevSecOpsDadAttack KQL Library](https://devsecopsdadattack.com/kql-library/) that demonstrate the same concepts in realistic detection and hunting scenarios.

## Absence Detection — `leftanti`

### SmartConnect Session Without Sign-In

**Presentation concept:**  
> `leftanti` tells you what is absent from the right-hand dataset.  
> It does not tell you what is absent from reality.

[View the full query in the KQL Library](https://devsecopsdadattack.com/kql-library/analytics-rules/detect-smartconnect-session-without-signin/)

A practical absence-detection example correlating successful SharePoint activity with sign-in telemetry using a composite user/IP key.

### VPN Session Without Prior Authentication

[View the full query in the KQL Library](https://devsecopsdadattack.com/kql-library/analytics-rules/detect-vpn-session-without-prior-authentication/)

An important counterexample: time-bounded absence cannot always be expressed safely as a simple `leftanti`. This analytic uses `leftouter` plus a windowed `countif()` to ensure the authentication event actually occurred within the relevant period.

---

## Detection Decay

### Analytics Rule Health

**Presentation concept:**  
> A quiet broken detection gets trusted.

[View the full query in the KQL Library](https://devsecopsdadattack.com/kql-library/health-checks/analytics-rule-health/)

Identifies Sentinel Analytics Rules that have executed successfully but produced no alerts during the review period.

A successful execution proves that the rule ran. It does not prove that the rule still provides meaningful detection coverage.

---

## `prev()` and Entity Boundaries

### Outlook Calendar C2 — Far-Future Standing Meeting

**Presentation concept:**  
> `prev()` knows rows. You know entities.

[View the full query in the KQL Library](https://devsecopsdadattack.com/kql-library/hunting/hunt-outlook-calendar-c2-far-future-standing-meeting/)

The query orders calendar events by mailbox, item, and timestamp before using `prev()`.

It then explicitly verifies that the previous row belongs to the same mailbox **and** calendar item before interpreting the relationship.

This is the practical version of:

```text
SORT → SERIALIZE → GUARD
```

### Day-by-Day Ingest Change

[View the full query in the KQL Library](https://devsecopsdadattack.com/kql-library/cost-and-ingest/day-by-day-change/)

A simpler contrast where the dataset represents a single ordered time series:

```kusto
| sort by TimeGenerated asc
| serialize previous_IngestedGB = prev(IngestedGB)
```

When there is only one logical sequence, `prev()` is simple. When multiple entities share the result set, you must establish and validate the entity boundary yourself.

---

## Representation vs Meaning

### Cloud Metadata SSRF — Normalized Forms

**Presentation concept:**  
> The string is not the thing.

[View the full query in the KQL Library](https://devsecopsdadattack.com/kql-library/hunting/hunt-cloud-metadata-ssrf-normalized-forms/)

Detects access to cloud instance-metadata services across multiple attacker-controlled representations rather than relying on one literal spelling such as:

```text
169.254.169.254
```

The query accounts for alternate IP representations, encoded separators, provider hostnames, wildcard DNS services, and other equivalent forms.

### Metadata IP — Inspect the DNS Answer

[View the full query in the KQL Library](https://devsecopsdadattack.com/kql-library/hunting/hunt-metadata-ip-any-encoded-form-inspecting-dns-answer/)

A complementary approach that moves the detection closer to semantic meaning: inspect what the hostname actually resolved to rather than trying to enumerate every possible way the attacker could write the destination.

---

## Process Relationships and Lineage

### npm Postinstall Grandchild Network Payload

**Presentation concept:**  
> Parent does not equal lineage.  
> If you care about ancestry, model ancestry.

[View the full query in the KQL Library](https://devsecopsdadattack.com/kql-library/hunting/hunt-npm-postinstall-grandchild-network-payload/)

Models npm lifecycle execution across multiple process generations, including patterns such as:

```text
npm
 └─ shell
     └─ curl
```

A direct parent-child test can correctly describe every individual process while still failing to describe the malicious relationship.

---

## Calendar C2

### Outlook Calendar C2 — Far-Future Standing Meeting

[View the full query in the KQL Library](https://devsecopsdadattack.com/kql-library/hunting/hunt-outlook-calendar-c2-far-future-standing-meeting/)

Operational hunting logic corresponding to the **“A Meeting in 2050”** example from the presentation.

### DNS AAAA Covert Recovery Channel

[View the full query in the KQL Library](https://devsecopsdadattack.com/kql-library/hunting/hunt-dns-aaaa-record-covert-recovery-channel/)

Companion hunt for Project CAV3RN's DNS AAAA recovery channel.

Together, the examples illustrate a broader lesson from the presentation:

> Your mental model controls where you look.  
> The attacker does not care which table you expected command-and-control to appear in.

---

## Full Library

The complete collection of hunting queries, analytics rules, health checks, and reference queries is available in the:

**[DevSecOpsDadAttack KQL Library](https://devsecopsdadattack.com/kql-library/)**
