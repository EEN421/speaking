# References & Further Reading

Supporting articles and research referenced throughout **The Query Ran Successfully. That's the Problem.**

These resources provide additional technical detail, examples, and background for several of the concepts discussed during the presentation.

## Detection Engineering & KQL

### The Dog That Didn't Bark

**Topic:** `leftanti`, absence detection, and the danger of interpreting missing rows as proof of missing activity.

- [KQL Detection of the Week: The Dog That Didn't Bark](https://devsecopsdadattack.com/2026-07-20-KQL-Detection-of-the-Week_-The-Dog-That-Didn't-Bark/)

**Referenced in:** Slide 5 — *leftanti: the dog that didn't bark*

---

### The Field That Wasn't There

**Topic:** Schema availability versus actual telemetry availability, field population, and validating data before building detection logic around it.

- [KQL Detection of the Week: The Field That Wasn't There](https://devsecopsdadattack.com/2026-08-24-KQL-Detection-of-the-Week-The-Field-That-Wasnt-There/)

**Referenced in:** Slide 8 — *The field that wasn't there*

---

### A Meeting in 2050

**Topic:** Calendar-based command-and-control, unexpected telemetry sources, and why an analyst's mental model can determine where they look for malicious behavior.

- [KQL Detection of the Week: A Meeting in 2050](https://devsecopsdadattack.com/2026-07-27-KQL-Detection-of-the-Week_-A-Meeting-in-2050-(Detecting-Project-CAV3RN's-Outlook-Calendar-C2-and-DNS-AAAA-Recovery-Channel)/)

**Referenced in:** Slide 11 — *A meeting in 2050*

---

## Representation & Evasion

### The String Is Not the Thing

**Topic:** Detecting semantic meaning rather than a single textual representation, including alternate representations of IP addresses, URLs, domains, paths, and other security-relevant values.

- [KQL Detection of the Week: The String Is Not the Thing](https://devsecopsdadattack.com/2026-09-01-KQL-Detection-of-the-Week-The-String-Is-Not-The-Thing/)

**Referenced in:** Slide 13 — *The string is not the thing*

---

### The Character Is Not the Payload

**Topic:** Unicode tag characters, encoded representations, and the importance of decoding and interpreting the underlying payload rather than detecting only the suspicious character.

- [KQL Detection of the Week: The Character Is Not the Payload](https://devsecopsdadattack.com/2026-09-08-KQL-Detection-of-the-Week-The-Character-Is-Not-The-Payload/)

**Referenced in:** Slide 15 — *The character is not the payload*

---

## AI, Automation & Security Engineering

### From RSS Noise to CISO Signal

**Topic:** Automating cyber-threat-intelligence collection, processing, prioritization, and analysis while preserving meaningful human judgment.

- [From RSS Noise to CISO Signal: Automating Cyber Threat Intelligence That Actually Matters](https://www.hanley.cloud/2026-04-28-From-RSS-Noise-to-CISO-Signal-Automating-Cyber-Threat-Intelligence-That-Actually-Matters/)

**Referenced in:** Slide 23 — *Now give the bad premise a GPU*

---

### AI Sovereignty on a Raspberry Pi

**Topic:** Local AI, model evaluation, and running LLM workloads with Ollama on resource-constrained hardware.

- [AI Sovereignty on a Raspberry Pi](https://www.hanley.cloud/2026-08-17-AI-Sovereignty-on-a-Raspberry-Pi/)

**Referenced in:** Slide 26 — *AI can challenge. Reality resolves.*

---

## Related Projects

Additional material related to the presentation and ongoing detection-engineering research:

- **DevSecOpsDadAttack:** <https://devsecopsdadattack.com/>
- **DevSecOpsDad:** <https://www.devsecopsdad.com/>

---

> **Presentation:** *The Query Ran Successfully. That's the Problem.*  
> **Speaker:** Ian Hanley  
> **Event:** KQL Café  
> **Topics:** KQL, detection engineering, telemetry validation, threat detection, security automation, and AI-assisted analysis