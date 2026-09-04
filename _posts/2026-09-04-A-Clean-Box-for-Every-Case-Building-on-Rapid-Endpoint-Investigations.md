---
title: A Clean Box for Every Case - Building on Rapid Endpoint Investigations
date: 2026-09-04
categories: [IncidentResponse, Tooling]
tags: [dfir, incidentresponse, cloud, kape, terraform, velociraptor]     # TAG names should always be lowercase
---

A case comes in late on a Friday. You've got a triage collection off a compromised endpoint and you need somewhere to work it.

So where does it go?

For most of us the honest answer is "the same analysis machine as always." Clear some disk space, unzip the collection, get to work. That's fine. It works.

It also means that when somebody eventually asks what else has touched that machine, you can't really answer. I'd lost count of the engagements on mine. The tooling was whatever version it happened to be whenever I last updated it. There was a volume still mounted from a case I'd closed months earlier because I kept meaning to detach it and never got around to it.

It is not a scalable solution.

So I built something else. It's a menu-driven console that spins up a brand new investigation host in the cloud for every case, mounts that case's evidence to it, and destroys the whole thing when the case closes.

This post is about why that's worth doing, and why I think it's worth forking even though it's rough in places.

## Where this came from

None of the methodology here is mine.

The whole approach (Velociraptor for collection, KAPE for parsing, Hayabusa for detection) comes from **Patterson Cake**, Director of Incident Response at Black Hills Information Security, and his Rapid Endpoint Investigations workflow. His [secure-cake/rapid-endpoint-investigations](https://github.com/secure-cake/rapid-endpoint-investigations) repo is the reference implementation I built the parsing side against. If you're only going to read one thing, read his stuff, not mine. The [Antisyphon course](https://www.antisyphontraining.com/product/workshop-rapid-endpoint-investigations-with-patterson-cake/) is worth the time and the [Rapid Endpoint Triage slides](https://www.blackhillsinfosec.com/wp-content/uploads/2023/12/SLIDES_Rapid-Endpoint-Triage-Patterson-Cake.pdf) are free. He also runs a four hour hands-on version through BHIS every so often.

What I added is the boring part around the outside. He answered "what do you collect and how do you parse it." I got stuck on a different question: where does that actually run, who pays for it, and what happens to the evidence when you're done?

The methodology was solid. My infrastructure for running it was a machine I kept around and hoped was clean.

## Why I built it

**Cross-contamination is a question you have to answer.** If HOST01 from one engagement and HOST02 from another have both been mounted on the same machine, that's part of your story now. Maybe nothing bad happened. You still have to explain it, and "nothing bad happened" is a much weaker sentence than "that machine had never seen another case." A host that gets destroyed at the end of a case makes the question go away instead of making you answer it.

**Drift makes your results unreproducible.** Parsers get updated, rule sets don't, and six months later the same collection gives you a different answer and you can't say why. The LOLBIN chain I walked through in [Seven Minutes](/posts/Seven-Minutes-A-Real-World-LOLBIN-Chain-From-Start-to-Finish/) leaned on prefetch, event logs and script artifacts, which are exactly the places a stale parser quietly changes what you see. A host built from the same definition every time doesn't drift.

**Not everybody has a forensics workstation sitting idle.** This is the one I actually care about. A dedicated analysis box per analyst is real money, and for a smaller team it's money that just doesn't get spent. Spin the box up when a case arrives, destroy it when you're done, and you're paying for the hours you were working. That puts a clean, properly tooled environment in reach of teams that were never going to buy one.

I'd been doing a manual version of this without noticing. When I dug into [CVE-2024-55591](/posts/Dissecting-CVE-2024-55591/) I stood the whole lab up by hand: vulnerable appliance, attacker box, tear it all down after. Same instinct, pointed at casework instead of research.

## What it does

Give the console a case ID, a cloud and a region. It builds three things with Terraform:

- Evidence storage (an S3 bucket or an Azure storage account), versioned, with optional WORM immutability
- An identity scoped to that case's storage and nothing else
- A Windows investigation host with no public IP

![The cloud console main menu, showing case workflow, shared infrastructure and teardown options](/assets/Images/Clean-Box-Cloud-IR/ir-cloud-console-menu.png)
_Arrow keys move, Enter selects, and typing the number or letter still works. Section headers can't be selected._

A collector built for that case uploads straight into that case's storage. When you connect to the host the evidence is already mounted as a drive letter and the toolchain is already there: the KAPE compound module, Hayabusa, Chainsaw, the EZ Tools suite. Same versions as the last case, because it came from the same definition, and it assembles itself while I'm gathering details about the case.

Close the case and one menu option destroys the compute and moves the evidence to cold storage. Another destroys everything and makes you type the case ID first.

The host never holds a credential. The evidence drive gets mounted using the host's own ambient cloud identity, an instance profile on AWS or a managed identity on Azure. No access key on the box, no SAS token sitting in a config file. If that host gets popped, the attacker can read the case it was already working on, and that's the whole blast radius.

And Terraform is never allowed to delete evidence. The storage module deliberately doesn't set `force_destroy`, so a careless `terraform destroy` physically can't take the evidence with it. It errors out instead. Emptying a bucket is a separate, deliberate act. Destroying evidence should require being explicit.

If the cloud side isn't your thing, the parsing console works fine on a laptop with a local collection:

![The IR console menu, showing setup, maintenance and parsing options](/assets/Images/Clean-Box-Cloud-IR/ir-console-menu.png)
_Same console pattern, no cloud account required._

## How this actually got built

I want to be straight about this, because it changes how you should read the code.

I didn't hand-write most of it. I vibe-coded a large chunk of this with an AI assistant. I described what I wanted, argued with it about the design, and tested what came out against real AWS and Azure accounts until it behaved. The architecture is mine. The calls about what gets stored where, what the host is allowed to reach, and what happens to evidence at the end are mine. A lot of the PowerShell and Terraform is not.

I don't think that needs an apology, but it does need saying, for two reasons:

First, calibrate your trust accordingly. This has been exercised end to end against live cloud accounts, including a real 17 GB collection, but it hasn't had the kind of review a tool you'd bet a case on deserves. Read it before you run it. That's true of anything you pull off GitHub and it's more true here.

Second, building it this way is the only reason it exists. I'm an incident responder, not a cloud engineer. The version of this project where I hand-write two clouds worth of Terraform in my spare time is the version that never gets finished. If you've been sitting on something similar because the infrastructure work looked like more than you wanted to take on, that wall is a lot shorter than it used to be.

## Why it's worth building on

It's not perfect. But the shape of it is right, and the shape is the part that takes the longest to get to.

**The methodology underneath is proven and it isn't hidden.** The parsing side references stock KAPE modules instead of reimplementing parsers, so when KAPE updates you get the update. There's very little clever custom code between a collection and your output, on purpose.

**It attaches to your network instead of creating one.** None of the Terraform modules build a VPC or a VNet. A case is disposable, your network isn't, and most orgs already have one they want these hosts to live in. That one decision is what lets it drop into an existing environment instead of demanding its own.

**The tool list is configuration, not code.** The analyst toolset is a JSON file. Ten entries, each with a tier, an enabled flag, and a field explaining why that tool is on the list at all. Add one, drop one, or change what a default build installs without opening a script.

**The cloud modules are small and they mirror each other.** Storage, identity and host are separate modules with the same shape on AWS and Azure. Want a third cloud, or want to swap S3 for something else? You're replacing one small module, not untangling a monolith.

**Nothing hides behind the menu.** Every console option calls a script you can also run directly with flags. The TUI is a convenience layer, not where the logic lives, so automating around it doesn't mean reverse engineering it.

**There's a self-test.** It runs offline pretty quickly, no cloud account and no credentials, and it checks the structural stuff: every menu option reaches a real function, every script it calls exists, every argument it passes is one the target actually accepts.

![Self-test output showing 151 checks passing](/assets/Images/Clean-Box-Cloud-IR/ir-selftest.png)
_Change something, run this, find out immediately whether you broke the wiring._

That last one matters more than it sounds. It's the difference between a repo you can read and a repo you can safely modify. I broke things on purpose to make sure it actually catches them, which I'd recommend to anyone who writes tests they've never seen fail.

It's MIT licensed. Take it apart.

## If you fork it

The KAPE compound module and the parsing scripts stand alone. If the cloud piece isn't interesting, ignore it, that half works on a laptop.

If the cloud piece is the interesting part, start in the storage modules. That's where the opinions live: retention, immutability, lifecycle to cold storage. Those are exactly the settings that should differ between organizations and they're isolated enough that you can change them without touching anything else.

And if you want to point this at a different collection format, the parsing side assumes a Velociraptor triage layout. That assumption lives in one place.

## Takeaways

For defenders and MSPs:

- A clean analysis host per case removes a question instead of giving you a better answer to it.
- Disposable infrastructure makes a properly tooled environment affordable for teams that were never buying a dedicated workstation.
- Ambient cloud identity beats stored credentials for evidence access. Nothing on the box to steal, nothing to rotate.
- Whatever you build, make destroying evidence an explicit act. Don't let a cleanup script be capable of it by accident.

If you're thinking about building something similar:

- Start from someone else's methodology. Patterson's work meant I never had to guess at what to collect or how to parse it, so I could spend the effort on the part that was actually missing.
- Build infrastructure that attaches to what exists rather than owning it. That's the difference between a tool people adopt and a tool people star and forget.
- Put the opinionated parts in config so people who disagree with your defaults can change them without forking your logic.

Repo's at [FLINTEK-LLC/ir-endpoint-investigations](https://github.com/FLINTEK-LLC/ir-endpoint-investigations). It's a framework, not a product. If you fork it and make it better I'd like to hear about it.

Most of what I learned here I learned by breaking it and going to look up why. That's fine. That's the job.
