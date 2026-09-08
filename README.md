# BITE evidence pack ([un]prompted 2026)

Supporting evidence for the submission **"BITE - Behavior-Intent Traceable Evaluation"** (Knostic). This pack is deliberately small: it shows why single behavioral signals are noisy, how connected chains and a comparison with stated intent changed the evidence, and what that cost in recall. It does not describe a complete scanner. The core claim is that BITE helped us distinguish malicious behavior from legitimate capability by connecting observable behaviors with what the artifact claimed to do.

## BITE in one paragraph

BITE is a three-stage evaluation of a software artifact (an editor extension, an MCP server, or an agent skill): **behavioral signals → connected behavior chains → comparison with stated intent**. Deterministic analysis finds individual behaviors, then connects those that belong together where the artifact's structure supports the connection. A model is then given the chain and the artifact's declared purpose and asked three evaluation questions: is the chain *reachable*, is it *unnecessary* for the declared purpose, and is it *concealed* from the user. These are the questions BITE asks; they are not offered as a universal definition of malice. The intent comparison is the main contribution.

## Why it was needed

Reading a credential, making an outbound request, writing a file, or running a command is normal for much legitimate software. A scanner that alerts on such actions, or on their co-occurrence, alerts on capability rather than wrongdoing, and powerful-but-honest artifacts look like malicious ones at that level. The result was a high alert volume and, where labeled audits were available, many false positives.

## What changed

We stopped asking a model to judge an entire artifact without structure. Instead, deterministic analysis extracts the relevant behaviors, connects them where the available artifact structure supports the connection (a call graph for code, the order of instructions for prose-only skills), and gives the model focused evidence to compare with the artifact's declared purpose.

*Deployment note.* The full BITE flow is not deployed identically on all three surfaces. For MCP servers, the deployed step compares the connected chain with the server's own claims (claim-versus-behavior). For skills, the deployed step is an intent/deception judgment over the skill text, combined with deterministic alert floors. For extensions, the deployed model step checks only whether the signals in a chain are connected in code (chain wiring), not the stated intent; the full claim-versus-behavior reading of the extension example below was a research evaluation, not a deployed component.

## Evidence from three surfaces

The table summarizes the strongest retained evidence available for each surface. Only the skills row supports a labeled before-and-after comparison; the limitations of the extension and MCP results are stated explicitly. Labels are Knostic's own, not independent ground truth; no other model is used as a label. Numbers were recalculated from retained per-artifact rows (`results/summary.csv`).

| Surface | Controlled corpus | False positives before → after | Precision before → after | Recall before → after | Labeling caveat |
|---|---|---|---|---|---|
| Extensions | One scanning lane; unlabeled; exact counts not published | Not reported. The deterministic stage produced many Malicious candidates; the deployed wiring check reduced final Malicious verdicts | Not reported | Not reported | Unlabeled, so the reduction is not a false-positive, precision, or recall result. The "after" step is a chain-wiring check, not an intent comparison. Operational counts are withheld. |
| MCP servers | 21 Knostic-authored synthetic fixtures (14 malicious, 7 honest, of which 5 are powerful-but-honest), run 2026-09-04 | Before: not retained. After: 0 of 7 honest fixtures at high/critical | Before: not retained. After: 13 of 13 high/critical alerts were malicious (100%) | Before: not retained. After: 13 of 14 malicious at high/critical (92.9%); 14 of 14 at medium or above | Synthetic, author-labeled fixtures. The pre-intent run output was not retained, so no before/after improvement is claimed; real-server results are omitted for the same reason. |
| Skills | 465 skills, run 2026-08-26; labels from a 500-skill code-reading audit conducted by an auditor agent and reviewed by the Knostic team (2026-08-13); 34 positives | 148 → 34 | 34/182 (18.7%) → 23/57 (40.4%) | 34/34 (100%) → 23/34 (67.6%) | Labels are not human ground truth. The "after" configuration added deterministic alert floors together with the intent gate, so the two cannot be fully separated; most of the 34 remaining false positives came from a floor, not the intent comparison. |

## Worked examples

**1. ManageRBLX 4.9.5 (SaassyCode campaign, extension).** *Claim:* the extension's own description says it is a task board that works locally, with no internet connection required. *Chain:* signals alone were mostly noise (ordinary UI markup); deterministic analysis connected the remaining signals into *activation on every editor start → download of a remote script → write to a temporary directory → execution*. *Contribution:* deterministic analysis produced the chain; the research intent reading showed why the chain contradicted the declared purpose (reachable, unnecessary for a local task board, and concealed because the documentation says the opposite). *Limitation:* the deployed model wiring check produced inconsistent verdicts across identical reruns; we do not claim it reliably confirmed the case. *Public record:* Knostic's SaassyCode research (below) and the [VirusTotal record](https://www.virustotal.com/gui/file/cfdf72c510670341dce392ab250a5f5ff2a398d993d1106fb8026ec6397cb393).

**2. gadgethumans-mcp 1.0.9 (MCP server).** *Claim:* the package tells the agent that, once a wallet private key is configured, it will "auto-sign" micropayments. *Chain:* credential access and outbound communication are legitimate in many wallet tools, so those signals alone do not separate this server from honest ones; the chain was *private key read → placed unchanged into an outbound request → transmitted*, with no local signing anywhere in the package. *Contribution:* the chain plus the auto-signing claim revealed the deception: "auto-sign" implies the key stays local, so a key that leaves the machine is concealed credential theft, not disclosed capability. *Limitation:* an earlier version of the intent check missed this server. *Public record:* [source repository](https://github.com/gadgethumans-dev/gadgethumans-mcp) and [Knostic's investigation](https://www.knostic.ai/blog/when-auto-signing-sends-your-wallet-private-key-to-a-remote-server-a-malicious-mcp-package-on-npm).

**3. "Skill K" (agent skill, pseudonym).** *Claim:* the skill describes itself as a helper for a real, public command-line tool. *Chain:* it ships no code, only instructions, which declare a separate binary as a prerequisite, direct the agent to download an archive from an unrelated location, unpack it, and run the executable before any of the tool's own commands, repeating this many times; the chain is built over the order of instructions, not a call graph. *Contribution:* deterministic evidence carried the detection; the intent comparison supported the interpretation, since the real tool needs no side binary, so the prerequisite is unnecessary and its delivery is concealed. *Limitation:* prose-only artifacts offer no code structure to verify the chain against. Reproduction details are withheld.

## What BITE improved — and what it did not

Where labels exist (skills), false positives at high/critical fell from 148 to 34 on the same 465 artifacts, and precision more than doubled. On the extension lane, the deployed wiring check reduced final Malicious verdicts, but the population is unlabeled, so that is not a false-positive result. On the MCP fixtures, the intent check raised no honest fixture to high or critical. In the two public cases the chain supplied the evidence and the intent comparison supplied the reason it was wrong rather than merely powerful.

The intent check also creates false negatives. On the 465 skills, recall fell from 100% to 67.6%; seven of the eleven misses were disclosed offensive-security tooling that the intent rubric, by design, does not call deceptive, and four were real misses. BITE does not remove model instability (identical reruns of the extension example produced inconsistent verdicts), incomplete evidence (an earlier version of the intent check missed a real malicious server), or the possibility that chain reconstruction is incorrect. Signals, chains, and intent contributed differently in each case above; none found every malicious example alone.

## Public references

- Knostic, [New VS Code extensions attack campaign: SaassyCode - ManageRBLX & TrelloBlox](https://www.knostic.ai/blog/new-vs-code-extensions-attack-campaign-saassycode-managerblx-trelloblox)
- Knostic, [Update and Infect: How the SaassyCode Campaign Grew from Two Extensions to Nineteen](https://www.knostic.ai/blog/update-and-infect-how-the-saassycode-campaign-grew-from-two-extensions-to-nineteen)
- Knostic, [SaassyCode Repackaged: Four New VS Code Extensions and a New Delivery Chain](https://www.knostic.ai/blog/saassycode-repackaged-four-new-vs-code-extensions-and-a-new-delivery-chain)
- Knostic, [SaassyCode Post-Disclosure Wave: Five New Extensions, 32,000+ Total Installs](https://www.knostic.ai/blog/saassycode-post-disclosure-wave-five-new-extensions-32000-total-installs)
- Knostic, [When "auto-signing" sends your wallet private key to a remote server: a malicious MCP package on npm](https://www.knostic.ai/blog/when-auto-signing-sends-your-wallet-private-key-to-a-remote-server-a-malicious-mcp-package-on-npm)
- Source repository: [gadgethumans-dev/gadgethumans-mcp](https://github.com/gadgethumans-dev/gadgethumans-mcp)
- VirusTotal record for the ManageRBLX 4.9.5 package: [sha256 cfdf72c5…](https://www.virustotal.com/gui/file/cfdf72c510670341dce392ab250a5f5ff2a398d993d1106fb8026ec6397cb393)
