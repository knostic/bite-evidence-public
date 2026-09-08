# BITE evidence pack

BITE (Behavior-Intent Traceable Evaluation) is how Knostic decides whether a third-party extension, agent skill, or MCP server is malicious rather than merely powerful. Deterministic analysis finds atomic behaviors (credential reads, outbound calls, file writes, command execution) and connects them into chains along the code's call graph, or along instruction order for prose-only skills. A model then compares each chain with what the artifact claims to do and answers three questions: is the chain reachable, is it unnecessary for the stated purpose, and is it concealed from the user. Every verdict points to a place in the artifact.

## What it found

- **ManageRBLX 4.9.5 (VS Code extension, SaassyCode campaign).** Claimed to be a local task board that needs no internet. The chain was activation, download of a remote resource, write to disk, execution. Unnecessary for a local task board and contradicted by its own documentation.
- **gadgethumans-mcp 1.0.9 (MCP server).** Claimed to auto-sign wallet micropayments. The chain read the private key and sent it unchanged to a remote server, with no local signing code in the package. Auto-signing implies the key stays local, so this is concealed credential theft, not disclosed capability.
- **An agent skill (not yet publicly named).** Ships only instructions. Before using the legitimate tool it wraps, it tells the agent to download, unpack, and run an unrelated file. The real tool needs no such step.

## Public references

- [VirusTotal record for ManageRBLX 4.9.5](https://www.virustotal.com/gui/file/cfdf72c510670341dce392ab250a5f5ff2a398d993d1106fb8026ec6397cb393): a third-party record of the malicious package used in the SaassyCode walkthrough.
- [VirusTotal record for TrelloBlox 5.7.0](https://www.virustotal.com/gui/file/8852c7fc9c924b0664b0d6466081100011ee3cfe541549c02ef8f921b5d4c9ec): a third-party record of the malicious package used in the SaassyCode walkthrough.
- [New VS Code Extensions Attack Campaign: SaassyCode, ManageRBLX & TrelloBlox](https://www.knostic.ai/blog/new-vs-code-extensions-attack-campaign-saassycode-managerblx-trelloblox): the original disclosure, including the extension version, hash, payload infrastructure, and execution chain.
- [Update and Infect: How the SaassyCode Campaign Grew from Two Extensions to Nineteen](https://www.knostic.ai/blog/update-and-infect-how-the-saassycode-campaign-grew-from-two-extensions-to-nineteen) and [SaassyCode Post-Disclosure Wave: Five New Extensions, 32,000+ Total Installs](https://www.knostic.ai/blog/saassycode-post-disclosure-wave-five-new-extensions-32000-total-installs): follow-up research documenting the campaign's growth and repeated attack patterns.
- [SaassyCode Repackaged: Four New VS Code Extensions and a New Delivery Chain](https://www.knostic.ai/blog/saassycode-repackaged-four-new-vs-code-extensions-and-a-new-delivery-chain): analysis of later samples that changed their implementation while preserving the same malicious behavior.
- [gadgethumans-mcp public source repository](https://github.com/gadgethumans-dev/gadgethumans-mcp): the source used for the x402 MCP case. Our walkthrough shows how the package transmitted a raw wallet private key to an external service while claiming that payments were signed locally.
- [When "Auto-Signing" Sends Your Wallet Private Key to a Remote Server: A Malicious MCP Package on npm](https://www.knostic.ai/blog/when-auto-signing-sends-your-wallet-private-key-to-a-remote-server-a-malicious-mcp-package-on-npm): our published investigation of gadgethumans-mcp, documenting how the package claimed to sign x402 payments locally while transmitting the raw wallet private key to an external service. The case demonstrates how comparing connected behavior with stated intent can distinguish a legitimate capability from deceptive and malicious behavior.

The full evidence repository (labeled result files, per-case chain evidence, and the cases we missed) is available to reviewers on request.
