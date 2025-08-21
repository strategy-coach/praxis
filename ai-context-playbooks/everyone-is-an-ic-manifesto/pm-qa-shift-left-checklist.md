# PM & QA Shift-Left Checklist

**Purpose:** Move from coordinators → ICs by producing executable inputs before development starts.

## Legacy Apps

**At Sprint Planning**

* [ ] Write **acceptance criteria as prompts** (include UI states, API payloads, error codes, accessibility rules).
* [ ] Provide **synthetic or anonymized datasets** (cover edge cases and compliance constraints).
* [ ] Draft **automated test starters** (e.g., Playwright/Cypress scripts, SQL assertions).
* [ ] Define the **“definition of done”** clearly in each ticket.
* [ ] Align test artifacts with stakeholders *before* dev picks up work.

## AI Apps

**At Sprint Planning**

* [ ] Publish **mock APIs / sample payloads** for early integration (simulate external systems).
* [ ] Specify **data cleansing rules** in natural language (include normalization, de-duplication, masking).
* [ ] Generate **persona-based chatbot transcripts** (angry customer, confused novice, fraudster, etc.).
* [ ] Create **bias / failure simulations** (multilingual, accessibility, schema drift, malicious input).
* [ ] Add **compliance guardrails** (HIPAA/PCI/GDPR checks) directly into acceptance criteria.

## Continuous Responsibilities

* [ ] Regenerate test cases as scope changes → **living UAT scripts**.
* [ ] Keep datasets and personas refreshed → **coverage doesn’t stagnate**.
* [ ] Pair with Dev/Data/Design → ensure inputs flow into CI/CD gates.
* [ ] Track telemetry from prod → fold back into prompts, datasets, and test cases.

**Reminder:** If PMs and QA do not attach prompts, datasets, and test starters to a story, it isn’t ready for dev. **Shift left = no merge without executable inputs.**
