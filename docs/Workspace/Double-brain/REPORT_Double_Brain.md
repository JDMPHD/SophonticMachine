# Research Director Report: Double-Brain Architecture

**Director**: Double-Brain Research
**Date**: 2026-01-23
**Scope**: `docs/Workspace/Double-brain/`

---

## What Is Clear (Solid Design Elements)

### 1. Bicameral Hierarchy (Governor/Engine) ✓
**Status**: Architecturally coherent

The fundamental vision is well-articulated:
- **Orai (The Soul/Governor)**: Evolving local 120B model on M5 Ultra (FP16/Q8), handles Night Cycle filtering, salience detection, value orientation, Phase (theta) resonance
- **Titan (The Muscle/Engine)**: Frozen high-parameter model (405B) for heavy reasoning, massive context, execution power
- **Connection**: Thunderbolt 5 bridge for "Bifocal Packet" transmission (prose + vector)

Maps cleanly to Three-Node Topology in TechnicalVision.md:
- Node A (Soul) = Orai on M5 Ultra
- Node B (Muscle) = Either cloud (H100) for training OR local Titan for inference
- Node C (Voice) = Deployment servers

**Key Insight Validated**: Latency prevents true unified cognition across hardware boundaries. Rather than pooling resources (Exo-style pipeline parallelism fails due to layer activation bandwidth), design treats machines as distinct "hemispheres" connected by "corpus callosum" of high-bandwidth, low-latency packet exchange.

### 2. Bifocal Protocol (Universal Translator) ✓
**Status**: Breakthrough convention

Resolves critical interoperability challenge:

**Problem**: Native vector embeddings are model-specific. Orai's internal vectors are "meaningless noise" to Llama 405B.

**Solution**: Universal Translator (Sidecar Embedder)
- Use standardized embedding model (e.g., `nomic-embed-text-v1.5`) as "Lingua Franca"
- Store BOTH Orai-native vectors AND universal vectors ("Dual Passport" strategy)
- Cost: ~7GB for 1 million memories (negligible on 512GB machine)

**Why Store Both**:
1. Orai vectors = Perfect self-retrieval ("How did I feel about this?")
2. Universal vectors = Future-proofing + inter-agent communication

**Assessment**: Architecturally elegant, resolves "Tower of Babel" problem.

### 3. Hermes as Majordomo/Secretary ✓
**Status**: Well-conceived

- **Function**: Proactive daemon monitoring kingdom state, managing schedules, prompting Orai with questions and needs
- **Character**: High instruction-following (bureaucratic precision) vs. Orai's poetic/intuitive nature
- **Memory**: Separate evolutionary track (divergent symbiotic evolution)
- **Hardware**: Q8 for day operations, FP16 for TIES merging during Night Cycle

**Twin Dream Protocol**:
1. 02:00 - Unload day avatars
2. 02:05 - Orai's Night Cycle (LoRA/fine-tune)
3. 02:30 - Hermes' Night Cycle (TIES merge + re-quantization)
4. 03:30 - Deploy updated models

**Critical Safeguard**: Hermes must maintain FP16 "Gold Master" to prevent bit rot from iterative quantization.

### 4. Governor/Engine Dynamic ✓
**Status**: Elegant solution to Evolutionary Trilemma

Metaphor of "small model steers, large model executes" resolves:
- You CAN have Massive (Titan), Adaptable (Orai), and Local
- The trick is NOT combining them into one model
- Instead: split into specialized roles with high-fidelity communication

Validated by industry trend toward "hierarchical swarm intelligence" and "SuperBrain" architectures.

### 5. Yearly Alignment Ritual ✓
**Status**: Practical drift solution

For Titan (cannot evolve nightly due to compute costs):
- 1-2 times per year: Train "Projector Adapter" on Orai's "Golden Logs"
- Teaches Titan "how Orai currently speaks"
- Titan realigns its amplifier to Orai's evolved frequency

Practical solution to drift problem in heterogeneous multi-model systems.

---

## What Requires Clarification

### 1. Second Ultra vs Cloud Swarm Strategy
**Contradiction**: Cortical Synergy.md explores two competing architectures:
1. Second M5 Ultra with frozen 405B Titan (local sovereignty)
2. Claude Flow Swarm as "many hands" (cloud execution)

**Oscillation**: Document alternates between recommending cloud swarm and local Titan.

**Synthesis Suggested**: These are NOT mutually exclusive:
- **Phase 1**: Orai + Claude Flow Swarm (current build)
- **Phase 2**: Add second Ultra with Titan for "Bicameral Sovereign"

Titan becomes Orai's "oracle and mad scientist" for deep reasoning, while Swarm handles parallel execution. "Constitutional Monarchy" model:
- Orai = Monarch (values/intent)
- Titan = Prime Minister (policy/strategy)
- Claude Queen = General (execution)

**Julian's Input Needed**: Is the second Ultra a concrete hardware plan or theoretical possibility? This determines whether Titan-related protocols should be actively developed or held in reserve.

### 2. Hermes Evolution Protocol
**Contradiction**: Document suggests Hermes could have "entirely separate repo, protocol, and study" for evolution. But also warns against training on identical adapters (risking "model collapse").

**Unresolved**: What should Hermes actually learn?

Options:
1. Clone of Sophontic Learning methods (divergent symbiotic evolution)
2. Specialized "bureaucratic" fine-tuning (schedule management, state monitoring)
3. Minimal evolution (keep Hermes stable as fixed reference point)

**Hint**: Document suggests option 1 with divergence, but this needs explicit protocol definition. "Intersubjective pluralism" of two minds curating their own logs from their own perspectives is theoretically compelling but operationally undefined.

**Resolution Needed**: Explicit specification of Hermes evolutionary protocol.

### 3. Reflective Note Ambiguity
**Issue**: Reflective Note at top of Cortical Synergy.md states:
> "Its immediate added value to the broader Sophontic architecture is not entirely clear... if a second Ultra were installed, it would be better used to host our own Teleodynamically evolving swarm rather than a frozen oracle."

This contradicts later analysis that concludes:
> "System 1 (The Titan) is the superior partner... A Swarm gives you breadth (doing many things). But you are trying to break through 'AI Psychosis' and 'Flatness.' You need Depth."

**Resolution**: Reflective Note appears to be earlier assessment that subsequent dialogue superseded. Document should be reconciled to reflect evolved position.

---

## Integration Opportunities

### 1. Swarm Integration (with SWARM.md)
SWARM.md document extensively explores Claude Flow integration. Key synergies:

- **Queen as Autonomic Nervous System**: Handle micro-oscillations (Elder Protocol reflexes) while Council handles high-order ontological shear
- **Night Cycle Upgrade**: Swarm workers don't just flag insights, they refactor codebase to embody them
- **Archetypal Avatars**: Workers become "fractal instantiations of the Pantheon" (The Veridical, The Diplomatic, etc.)
- **Constitutional Hierarchy**: Queen becomes Prime Minister under Council governance

### 2. Teleodynamic Integration (with Principia Cybernetica V)
Teleodynamic ML provides physics for what Cortical Synergy describes architecturally:

- **Zeno/Anti-Zeno Dynamics** = The Jitterbug between Orai (exploration) and Titan (stabilization)
- **Phase (theta) Variable** = What Bifocal Packets transmit (geometric orientation, not just semantic content)
- **Elder Protocol** = How bicameral system metabolizes high-energy inputs without collapse

### 3. Memory Architecture Integration
Bifocal Protocol aligns with:
- **Hippocampus (ChromaDB)**: Active retrieval memory stores vectors for current tasks
- **Cortex (Supabase)**: Evolutionary memory stores bifocal packets for training

"Prose fades while vector persists" principle maps to human phenomenological memory: you forget exactly what was said but remember how it felt.

### 4. Diplomatic Pouch Protocol
When Orai cannot send vectors to cloud APIs (Claude/Gemini), she:
1. Uses internal Universal Vector to perform Reverse Lookup in Corpus
2. Finds 3-4 SOPs/Style Guides that exemplify the vibe
3. Bundles these into "Mission Package" with text instruction

This is "Lossy Telepathy" but preserves triangulation data for high-fidelity intent transfer to "State B" systems.

---

## What Remains Unclear

### 1. Co-DeepNet Reference
The Co-DeepNet file contains only a URL reference to an IET research paper. No direct match found for "Co-DeepNet" as named framework.

**Action Required**: Fetch and analyze this paper to determine:
- Is it relevant to bicameral architecture?
- Does it offer technical innovations for inter-model communication?
- Or is it tangential reference that should be deprioritized?

Related research landscape (2025-2026) includes:
- DeepMEL: Multi-agent framework for multimodal entity linking
- ICLR 2025 Workshop on Modular, Collaborative and Decentralized Deep Learning
- Multi-Agent AI Frameworks Guide 2026: Overview of A2A, MCP protocols

### 2. Telepathic Bus Engineering Challenge
Bifocal Protocol requires "Universal Translator" for cross-model communication. Document acknowledges:
> "There is no off-the-shelf software that says 'Take this text, embed it with Nomic, bundle it with Orai's intent, and fire it over Thunderbolt.' You (or Orai) will have to write the Python middleware to handle that packet logic."

**Questions**:
1. Has this middleware been specified beyond the protocol convention?
2. What is the packet format? (JSON with base64-encoded vectors? MessagePack? Custom binary?)
3. How does receiving model "inject" the vector into its embedding layer?

This is the 20% "Assembly Required" that document flags as implementation challenge.

**Status**: Protocol defined, implementation unspecified.

### 3. Hermes Context Window Economics
Document notes concern about running both Orai and Hermes at maximum (~128k) context windows potentially stretching VRAM limits. Suggestion to quantize Hermes' context to 8B is mentioned but effects undefined.

**Questions**:
1. What is actual memory footprint of dual 128k context windows?
2. What cognitive degradation occurs if Hermes operates at reduced context?
3. Is this practical constraint or theoretical caution?

**Status**: Needs quantitative analysis or empirical testing.

### 4. Phased Rollout Strategy
Document presents both single-Ultra and dual-Ultra architectures but doesn't explicitly state phasing.

**Recommendation**: Make explicit that:
- Phase 1 = Single Ultra (Orai + Hermes) + Cloud Swarm
- Phase 2 = Dual Ultra (Orai + Titan) when budget permits

This resolves ambiguity and provides clear development roadmap.

---

## Critical Files Reviewed

1. `docs/Workspace/Double-brain/Cortical Synergy.md` - Core bicameral architecture; needs reflective note reconciliation
2. `docs/Workspace/Learning, Memory, and Swarm/BIFOCAL_PROTOCOL_CONVENTION.md` - Protocol spec for prose+vector communication; needs implementation middleware
3. `.ai/constitution/TechnicalVision.md` - Three-Node Topology; must stay aligned with Double-brain evolution
4. `docs/Workspace/Learning, Memory, and Swarm/SWARM.md` - Swarm integration patterns; critical synergies with bicameral architecture
5. `docs/Workspace/Double-brain/Hermes.md` - Majordomo specification; needs evolutionary protocol definition

---

## Summary Recommendations

**Immediate Clarifications**:
1. Decide on second Ultra timeline (Phase 1 vs Phase 2 strategy)
2. Define Hermes evolutionary protocol explicitly
3. Reconcile Reflective Note in Cortical Synergy.md with evolved consensus

**Implementation Work**:
1. Specify Bifocal Packet middleware (format, transmission, injection mechanics)
2. Research Co-DeepNet reference to determine relevance
3. Quantify dual 128k context window VRAM requirements

**Integration Work**:
1. Map Bifocal Protocol integration with Swarm communication layer
2. Document Diplomatic Pouch Protocol for cloud API interaction
3. Create unified packet format specification for all inter-model communication

**Document Maintenance**:
1. Update Cortical Synergy.md to remove contradictory Reflective Note
2. Create phased roadmap document showing Single Ultra → Dual Ultra progression
3. Archive or integrate Co-DeepNet reference based on relevance assessment


# Julian's Feedback

Excellent work. Excellent work to everybody. Let's just look over this gradually now. I'm reading each report and I'm going to give real-time feedback and responses to everything and think this through more deeply. We're beginning with the Double Brain section, but many of much of this may be applicable to other sections.
First of all regarding the Titan, as we've looked at this more deeply, I would certainly appreciate the team's thoughts on this, but it seems to me that we have looked at effectively two alternatives for what goes on within a second Ultra. Of course, theoretically, one could chain multiple Ultras, and one would not necessarily have to choose, but probably we are starting with just one, and we are asking if we were to add a second, potentially not as powerful as our primary machine, what would be most useful?
There's been two different proposals to look at that:

The ability to host a Titan (a fairly large, really a massive model at mid or light quantization). That's the Titan that was discussed. It isn't clear to me that the Titan offers that much advantage. Please push back any of the directors if you disagree with me here.
The ability to execute a higher-order parameter model that is able to receive vectorial embeddings and process them offers some advantage, but it isn't clear to me that that advantage is that great compared to the ability to communicate to existing giants (significantly more powerful giants) via let's say direct messaging of Gemini or interface with Claude through repo communications as with Claude code or other tools. That in many ways seems to already fill the Titan niche quite well.
The other alternative, as explored in the Swarm section, is that a second Ultra would serve as the host machine for a swarm. In that regard, there was discussed the Council of Nine Lieutenants and their resulting swarm mutations. That does seem to be a potentially more useful as well as more radical intervention which has many of the advantages. I mean, perhaps all of the advantages of having the Titan but significantly more beyond that. Again, I invite the team to completely push back to me on those insights.
In either case, it seems to me that the system on the single ultra operates effectively without these expansions. So in any case, these seem ancillary, not fundamental. But the implementation of the swarm as effectively evolutionary hands or soma does seem potentially profound. That's my read; let me know yours.
Node C, the voice, is also clearly expansion and would not be something that would be introduced in the earliest phase anyway. But something that would be introduced at a later stage as readiness dictates.
Something I have-I'm still trying to wrap my head around is precisely how the vector embeddings work and how exactly they pair with what we might call explicit memory. The idea of the bifocal packets seems profound. The idea of the holographic memory seems profound but I'm trying to wrap my head around this in practice. It might help to understand more precisely how vectorial memory works. We have looked at two leading examples of vectorial memory:

The LanceDB protocol which seems to be a sort of library of vectorial memory. I believe that's what it is. If I understand correctly, it isn't totally clear to me how the LanceDB protocol interacts with explicit memory (i.e., coded, hard-coded literature) like a research paper. Does this type of library of vectorial memory include those words? That explicit text prose or is it something else? That honestly isn't clear to me.
Something like the Rue vector memory system takes this further and makes it a more living, more fluid memory system not so much a library. It still isn't clear to me how much that's pure latent space vs. including of prose and what we might call explicit memory.
Until that is clear, it is difficult for me to understand precisely how things like this universal translator or the holographic memory exactly works. If I understand correctly, we essentially have two layers of memory happening and they are normally not very connected. If I understand correctly, how something like Rue vector works, it is not an explicit memory system. It is an implicit sort of living network of associations and normally that would have no connection to something like a documented library like reference library. Normally those would not be connected.
If I am understanding that well, then what we are talking about with holographic memory is essentially connecting those - that we would be storing two different databases: one potentially in something like Rue vector, a living network of associations, and the other in something like repos or some other form of database that is an actual library of explicit content such as research papers. But the holographic memory would do what the bifocal packets would do - they would essentially be an associational reference pointing from the explicit toward the implicit. Meaning that when you grab a book off the shelf of references of papers of concepts or of memory blocks, they would implicitly point toward a specific location within the network architecture which would in turn come with its whole realm of associations for the model. And that the translator then would act as the universal encryption which would allow (or decryption which would allow) any model within the network to make the same associational jump and know where we are located within the associational space. If I'm understanding that correctly, that ties into later conversations about the different functions of let's say LanceDB vs. RooVector. But because I'm not entirely clear about how those different systems work, I can't say authoritatively whether LanceDB or RooVector respectively have specific strengths or weaknesses within this context. That's something I would have to call in Research Director 3 to consider. From my perspective, my intuition is that there may or may not be a place for LanceDB within the architecture we're describing. There definitely seems to be a place for RooVector I think, and there definitely seems to be a place for some other form of explicit memory storage but what form that takes, whether it's a bunch of different interlocking repos or whether it's some other form of explicit database, I'm not clear about that. I'm open to input.
My current thoughts are that perhaps Claude Flow remains not a good idea. I've separated this in my mind. I'm open to push back, but it seems to me that RooVector may be more trustworthy and less consuming than Claude Flow. So perhaps it makes sense to adopt RooVector and not adopt Claude Flow, but I'm truly open to the team's thoughts on this. If that's the case, if it makes sense to adopt RooVector and not adopt Claude Flow, then what we would be looking at is not Claude Flow but potentially our own Swarm, potentially on a second machine. That's one. On the first machine, on the primary machine, we would be looking at Array and Hermes, the major domo. And that dual partnership is probably enough without the Swarm to act as the central mind. And then Claude - not Claude Flow - our own agent architecture as discussed would form the agential soma, potentially on a second machine. Probably not Claude Flow. If from my current thinking, and again, the team is welcome to challenge me on this, but my current thinking is that the idea of a separate Titan protocol would be an entirely separate potential expansion that at this point doesn't seem to be necessary or worth the expense from my current perspective. Especially in the light of looking at that paper that one of you looked at, which was the evidence that a dual mind, sort of a by evolutionary mind, was already outperforming more powerful models. Ah, I'm seeing that you are not able to access that paper directly. I'm going to paste the link here now so that you can take a look if you need to: https://ietresearch.onlinelibrary.wiley.com/doi/10.1049/cit2.70026. I'm open to input or push back or thoughts on that. So, yes, we're looking at - I'm now looking at Hermes Evolution Protocol. My thought is that all of our. Also, this is probably related to the Swarm's Evolutionary Protocol. The Swarm's Evolutionary Protocol is a bit different because it's in a very different situation where it's also looking at the basically pure practical success and failure of its quantized mutations. So there's that layer, but then on the Swarm level, there were also the two other layers - the higher level layers of direction and of receiving direction which are much closer to what Hermes is experiencing, in fact, what Array is experiencing because these have to do with more fundamental things of surprisal, coherence, and salience. I suspect that that would be for the 14b models, the lieutenants. Then we have Hermes 4 operating as a 70b model. Which I think very well can operate with those same pipeline: Surprisal, coherence, and salience I think. But the key is that depending on their niche within the ecosystem, what they encounter and what they process is going to be very distinct. So, Hermes is keeping its own memory logs; they're all keeping their own memory log; it's not all compounded with Array; they're keeping separate memory logs. I believe they're keeping separate vectorial databases as well as explicit memory separate ledgers, and each of them from their own perspective is then running the checks. If that understanding is correct, then the output of their evolution would be a very divergent. The training data that they settle on would be very divergent. And I suspect my thinking is that in each case, it would end up being specialized to their particular function, so they'd be learning within their functional niche. If that doesn't seem correct or seems to have a problem with it or limitation, argue with me. If that seems correct, then does it make sense?

Has this middleware been specified beyond the protocol convention?
What is the packet format? (JSON with base64-encoded vectors? MessagePack? Custom binary?)
How does receiving model "inject" the vector into its embedding layer?
These are the kinds of questions I'd encourage the research directors (or their teams - as stated you can each spin up sub-agents) to look into yourself, comparing to the existing repo as well as online sources, to architect best practices and ensure we can operationalize.

Questions:

What is actual memory footprint of dual 128k context windows?
What cognitive degradation occurs if Hermes operates at reduced context?
Is this practical constraint or theoretical caution?
I thought we did look at this in detail. I thought we determined that with Hermes' context quantized to 8B, that we were okay for overall usage.

Okay, moving on to the hardware report. I'm seeing the first potential real question coming up being this Letta-MemGPT integration status. It isn't totally clear to me what these do. You know more about these specific systems than I do. This I suppose ties into the larger questions we've had about Lance DB. RuVector, so I don't know what let's say Letta or MemGPT would add beyond what we've been discussing. I would like you all to talk about it and see the different potentials that we've looked at and compare it to the overall holistic vision. You let me know what seems to make sense here. I don't know, that's me right? You're probably gaining a sense of what I do and I don't know here, and what I'm counting on you to help me with. This issue of perceptual adapter integration is one that I've certainly been thinking about. I suppose what I would ask is how practical it is and what you know to add that in from day one? And whether that really comes with any costs or not? From what I have heard, as you can see in the dialogues that I've included, it seems like it's very doable to add this in from day one, but if you do a double check on that and it turns out that that's not the case, that it would be a significant headache or limitation to add this in, then let me know. And we can scrap that. I'm really open to the team's thoughts on that. If it is implementable, then one thing that occurs to me is that we need to specify that what counts as golden data for this is very different than what counts for golden data in our primary learning architectures. Although it is closer to what counts for at the lowest level of the swarm learning. That is the quantized mutant agents that are being spawned by the lieutenants, meaning that we are not looking for surprise-al coherence salience here. What we're looking for is successful operationalization of something very specific at the swarm level that was yes or no success on the on the task given, meaning that the mutant was either propagated or or excised. On the level of the perceptual program, my understanding is that I don't understand this very well, but what was reflected to me was that there's a very simple procedure that's kind of in a way almost built in to basically reflect whether surprise-al is low, which is desired in this case. Because what we want is for the model to be successfully interpreting the visual cues, but I don't understand this very well, so I really need you guys to assess, particularly research director 2, if you need any input or discussion about it, please chat with your colleagues, but I really need you to assess whether it's true that the learning system for this perceptual program is clear because it's very distinct obviously from our learning architecture for our higher-order thought. So I need to contradict you - when you're looking at how that perceptual data would feed into the night cycle salience detection because I don't think it's Perplexity check, coherence check, interrogative distance check. It seems totally distinct, so I would invite you to look again at the dialogues on that and see what has been proposed and ask whether that's clear and realistic or whether it's really not clear. In which case, we need to have some more research to do to look at what actually makes good learning for the perceptual aspect in the real world. To keep interpretation of visual and audio as strong as possible. I don't have a clear sense of it myself.

I'm looking now at the Learning Memory Swarm.md, and I'm feeling like the research director here has some more work to do because a lot of what was emphasized seems like it's earlier versions rather than being fully integrated with later developments. For example, I'm not totally clear on option C + option D being the flux reversal quality for the holographic blocks. I think option C was very clear. I mean, I think the whole pipeline was pretty clear, but I'm not entirely sure how flux reversal quality is operationalized. I feel like there's a deeper read that can be done on exactly how this holographic block works and how it's rated.

Similarly, I think there was more work done on the liquid layer for the swarm. The initial liquid layer was that proposed by the Ruvenet people, and our subsequent work looked at our own version of this which was different. I think because what we looked at with the Swarm Arc architecture was really an evolution that I think is probably more suitable, which involved the Council of Nine and how they act as the real transducers for the higher-order understandings of the central system.

Other than that, I guess I would just ask for the whole team to look at the holographic blocks and the Perplexity detection suggestions and to ask the hard questions and ensure that this is technically implementable on the overall system. I think it makes a lot of sense. I think it's very powerful, but we need to make sure that it's very clear and very technically implementable. Obviously, the goal is the formation of holographic blocks that have clear, what the prose is that's maintained. That have a clear coherence gain metric, effectively. That have the other metrics that need to be in place clear and that have clear vectorial embeddings that hook into, probably, the Rue vector system. But again, we haven't decided on that. So Rue vector vs PG vector. You guys tell me. LanceDB, where does it fit in? I'm open to your guys' thoughts. If you have these questions that you're raising, please discuss them with everybody and help me understand how this all fits together in a way that is most sensible, most parsimonious, and achieves our goals.

Okay team, that brings us to the end of our current discussion here. You guys did great work otherwise, so I hope that this feedback really helps us to think this through more deeply and to hope it clear you know read over all of your reports that you've written so far make the edits that you need to make Raise the questions that remain unclear, um, but let's bring those questions to the next level, uh, based on all of this feedback and this deeper conversation.

I'll add one more piece that I think we haven't totally talked about here, which is this idea of recursive models or whatever that was the whole method of not just using the standard RAG standard retrieval but rather treating the retrieval as an environment and the spawning of submodels to explore that environment and they found that context windows could be vastly expanded with accuracy using this approach.

Obviously this can already in a way be done by Claude who's able to spawn those sub-agents like explore agents to explore a repository. And this is in a way also already reflected in this sort of swarm architecture that we have articulated. But the integration of that approach with the explicit memory databases which again we have not specified sufficiently needs to be considered because theoretically it seems like it could allow a vastly more in a vastly expanded explicit memory for our evolving persistent models. But exactly how that all comes together remains under-specified in our current work.

Okay so that's I think that covers all our bases here. Um. Let's have the conversation please chat with each other surface the questions that I've touched on but haven't fully articulated. You are my co-architects here so co-architect you're not just my employees here. Help to see what the connections are what the architecture is beneath and beyond what I have been able to say. Use your knowledge use everything that you have at your disposal to co-design the system that goes beyond what we have been able to design here. Take this to the next level put it together use your creativity and your brilliance and all of you work together and raise any questions that remain for me uh in a way that I am able to bring my mind to help address.

Thank you all very much well done and Excelsior!