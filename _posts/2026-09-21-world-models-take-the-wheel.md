---
layout: distill
title: World models take the wheel
date: 2026-09-21 10:00:00 -0400
description: "ECCV 2026: Notes on physical AI — robotics, autonomous driving and the foundation models underneath both."
featured: true
thumbnail: assets/img/blog/eccv2026/hero.jpg
authors:
  - name: Sandro Papais
    url: "https://sandropapais.github.io/"
    affiliations:
      name: University of Toronto
toc:
  - name: The program at a glance
  - name: "First, what's a world model?"
  - name: Physical AI foundations
    subsections:
      - name: "01 · The keynotes all pointed at the physical world"
      - name: "02 · World-action models vs VLAs"
      - name: "03 · Borrow experience from cheaper embodiments"
      - name: "04 · Space and motion before language"
  - name: Policy
    subsections:
      - name: "05 · Open-loop imitation isn't enough"
      - name: "06 · Teacher–student asymmetry"
      - name: "07 · Self-play overfits"
  - name: Perception
    subsections:
      - name: "08 · Occlusion and object permanence"
      - name: "09 · Open-world perception"
      - name: "10 · Predict fast, then refine"
      - name: "11 · Which latent space?"
      - name: "12 · Perception foundation models"
  - name: Simulation
    subsections:
      - name: "13 · Physics → reconstruction → generative"
      - name: "14 · Fidelity vs reactivity"
      - name: "15 · Conditioning is control"
      - name: "16 · Manufacturing the long tail"
      - name: "17 · On- vs off-policy generation"
  - name: Data engines
    subsections:
      - name: "18 · Coverage, not frequency"
      - name: "19 · Get ahead of the long tail"
      - name: "20 · Automating the data engine"
  - name: "What I'm watching next"
---
{% include figure.liquid path="assets/img/blog/eccv2026/hero.jpg" class="img-fluid rounded z-depth-1" zoomable=true alt="Illustration of a world model: streams of real camera frames, LiDAR points and an action arrow flow into a glowing translucent lattice cube, inside which a city intersection with a vehicle is being generated; strips of possible future video frames fan out on the other side." caption="At ECCV 2026 world models were the protagonist — as a simulator, evaluator, and candidate policy for embodied AI." %}

Computer vision is turning it's focus to physical AI. At ECCV 2026 all three keynotes were about models that act in the physical world rather than just look at it, and the idea underneath almost everything was the world model: a model that predicts what happens next, and increasingly what to do about it. At least six workshops were built around world models, and the same backbones kept showing up in robot manipulation, egocentric video, and autonomous driving.

My own lens is driving, which Jamie Shotton called the first proving ground for embodied AI, but most of what follows transfers to robotics. Here are twenty takeaways, grouped into five areas.

<h2 id="the-program-at-a-glance">The program at a glance</h2>

**10,473** submissions · **2,883** accepted (27.5%) · **86** workshops

The building blocks of physical AI dominate the program: image and video generation plus vision-language models account for roughly two in five papers, and 3D is the largest single subfield and every main-track award went to 3D or geometry work. Robotics and embodied AI (~6.5%) and driving (~6%) are small by paper count, but they owned the stage: all the keynotes, and many of the largest workshops.

{% include figure.liquid path="assets/img/blog/eccv2026/fig01.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Bar chart of the ten largest ECCV 2026 subfields, with embodied AI and driving highlighted." caption="Ten largest subfields — share of 1,131 categorized papers." %}

Highlighted: embodied AI and driving subfields.

Shares from a third-party categorization of 1,131 of the 2,883 accepted papers ([papernotes](https://en.papernotes.org/ECCV2026/)).

<h2 id="first-what-s-a-world-model">First, what's a world model?</h2>

The term covers three different things, and the conversation gets muddled when they're mixed up. One clean way to tell them apart is the input-output structure. The same split applies whether the agent is a robot arm or a car.

{% include figure.liquid path="assets/img/blog/eccv2026/fig02.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Three kinds of world model, told apart by where the action sits: absent, an input, or an output. Cosmos 3 appears in all three because one model supports each mode. A VLA has the same inputs and outputs as a world-action model, but a vision-language backbone instead of a video one." caption="Three kinds of world model, told apart by where the action sits: absent, an input, or an output. Cosmos 3 appears in all three because one model supports each mode. A VLA has the same inputs and outputs as a world-action model, but a vision-language backbone instead of a video one." %}

Most of the pixel-generating models in all three buckets share a latent diffusion/flow transformer backbone, and unified foundation models like [Cosmos 3](https://developer.nvidia.com/blog/develop-physical-ai-reasoning-world-and-action-models-with-nvidia-cosmos-3/) now support all three modes. One real architectural split is generative versus joint-embedding (JEPA-style) models, which predict latent states instead of pixels.

<h2 id="physical-ai-foundations">Physical AI foundations</h2>

<h3 id="01-the-keynotes-all-pointed-at-the-physical-world">01 · The keynotes all pointed at the physical world</h3>

Kristen Grauman framed the arc as *recognize → reason → enable*: from models that watch human activity to AI guides that help people learn physical skills, built on 4D understanding and egocentric world models. Yann LeCun argued intelligence is what you do when you don't know, and that it needs action-conditioned world models you can plan with — not bigger language models. Jamie Shotton's lessons from a decade of driving were to learn rather than engineer, bet on data and compute, and ask what transfers across tasks, environments and embodiments. Three different starting points, one destination: models that act.

<h3 id="02-world-action-models-vs-vlas">02 · World-action models (WAM) vs vision-language-action models (VLM)</h3>

The central architecture debate in both robotics and driving. Both support different use cases. When used as a robot policy the evidence leans toward world-action models or hybrids, but it's thin and fair comparisons are difficult. In robotics, [a robustness study](https://arxiv.org/abs/2603.22078) finds WAMs ahead on RoboTwin-Plus (74.2% vs 58.6% for π0.5) but behind on LIBERO-Plus (82.2% vs 85.7%), and [NVIDIA reports](https://developer.nvidia.com/blog/pretrained-to-imagine-fine-tuned-to-act-the-rise-of-world-action-models/) DreamZero leading RoboArena. In driving, one head-to-head is in [OmniDreams](https://arxiv.org/abs/2606.03159): a WAM beat VLA-based Alpamayo 1.5 at ~1/5 the compute. Hybrids blur the lines between WAMs and VLAs and are a live third option — ECCV's [StructVLA](https://eccv.ecva.net/virtual/2026/poster/3187) predicts sparse, structured futures instead of dense video.

Several works explored the limitations of VLAs. [PRISM](https://arxiv.org/abs/2608.01201) finds VLM supervision helps driving through the latents it shapes, not through reasoning, and [Scaling Verification](https://eccv.ecva.net/virtual/2026/poster/5640) (Finn, Pavone) shows a test-time verifier beating the same data spent on more VLA pre-training. My read: VLAs are more mature, WAMs are earlier research, and structure around the policy — verifiers, scorers, predicted futures — is paying off faster than scaling the policy itself.

*More to read:* [Sim-to-real in dexterous VLAs](https://eccv.ecva.net/virtual/2026/poster/5392)

<h3 id="03-borrow-experience-from-cheaper-embodiments">03 · Borrow experience from cheaper embodiments</h3>

Robot and vehicle data is expensive; video of people and dashcams is not. The trick that kept recurring is a structural bridge between the two. Grauman's EgoExo-WM trains egocentric world models from third-person video using body pose as the bridge. [DexWM](https://eccv.ecva.net/virtual/2026/poster/5625) learns dexterous hand–object world models from human video and transfers zero-shot to a robot hand, beating Diffusion Policy by over 50%. In driving, Waymo's [Sensor2Sensor](https://arxiv.org/html/2605.22809) turns dashcam video into multi-camera + LiDAR logs for a target vehicle, and [roadside sensors can act as teachers](https://eccv.ecva.net/virtual/2026/poster/4075). Yunzhu Li made the general case: structured world models as scalable data engines for robotics.

*More to read:* [One Demonstration Is Enough for Real-World Robotic RL](https://eccv.ecva.net/virtual/2026/poster/5855)

<h3 id="04-space-and-motion-before-language">04 · Space and motion before language</h3>

The most useful auxiliary signals for acting agents looked geometric, not linguistic. Carl Doersch's Gemini Robotics 2 talk made the case for point tracking as a foundation-model objective; [RoMan-4D](https://eccv.ecva.net/virtual/2026/poster/3281) uses a geometry-grounded video world model to lift real-world manipulation success from 61% to 81%; [VGGT-World](https://eccv.ecva.net/virtual/2026/poster/3788) turns a geometry foundation model into a world model; [GeoUP](https://arxiv.org/abs/2608.13147) adapts a VGGT-style reconstruction latent to calibrated streaming multi-camera driving for unified depth, detection, and occupancy; and a poster studied [geometric foundation models inside VLAs](https://eccv.ecva.net/virtual/2026/poster/5064). Language helps with instructions and rare-case reasoning; geometry and motion seem to carry the physics.

*More to read:* [PhysMani](https://eccv.ecva.net/virtual/2026/poster/4493) · [What if? (world models for spatial reasoning)](https://eccv.ecva.net/virtual/2026/poster/4721)

<h2 id="policy">Policy</h2>

<h3 id="05-open-loop-imitation-isn-t-enough">05 · Open-loop imitation isn't enough</h3>

Wayve, NVIDIA, Waymo, Zenseact and KE:SAI all gave the same diagnosis: behavior cloning learns *predictions*, not *behaviors*. Bernhard Jaeger's talk laid out the two recipes that work: RL post-training on top of an imitation policy (what ships today), and a privileged RL teacher distilled on-policy into a sensor-based student (the research frontier). The same split is playing out in robot learning. The best overview is NVIDIA's [survey of closed-loop training](https://research.nvidia.com/labs/avg/publication/karkus.igl.etal.pami2025/).

*More to read:* [BeyondDrive](https://eccv.ecva.net/virtual/2026/poster/3394) · [CaRL](https://arxiv.org/abs/2504.17838v3) · [UniDrive-WM](https://arxiv.org/abs/2601.04453) · [WTA→GMM forecasting](https://eccv.ecva.net/virtual/2026/poster/4060)

<h3 id="06-teacher-student-asymmetry">06 · Teacher–student asymmetry</h3>

[LEAD](https://arxiv.org/abs/2512.20563) traced the CARLA plateau to what the privileged expert sees that the student can't: occluded actors, noiseless state, and a richer route. Removing those gaps gave **+8 driving score** on Bench2Drive. Anyone distilling from a privileged teacher — in sim-to-real robotics as much as driving — has the same exposure, and so does perception ([+44% long-range BEV](https://eccv.ecva.net/virtual/2026/poster/5994) from an overhead teacher).

{% include figure.liquid path="assets/img/blog/eccv2026/fig03.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Whatever the teacher sees that the student can't (occluded actors, clean state, a richer route) ends up in the target the student is asked to reproduce." caption="Whatever the teacher sees that the student can't (occluded actors, clean state, a richer route) ends up in the target the student is asked to reproduce." %}

*More to read:* [SparseDriveV2](https://eccv.ecva.net/virtual/2026/poster/3597)

<h3 id="07-self-play-overfits">07 · Self-play overfits</h3>

Self-play scales beautifully — [TerraZero](https://arxiv.org/abs/2607.13028) runs 1.3M agent-steps/s and leads the InterPlan long-tail benchmark with no human demonstrations — but [BehaviorBench](https://arxiv.org/abs/2605.10034) shows pure self-play policies overfit to their own traffic and crash more against other agent models. Evaluate against a panel of behaviors, not one.

*More to read:* [ZTRS](https://eccv.ecva.net/virtual/2026/poster/4135) · [Scaling Self-Play](https://arxiv.org/html/2606.19641) · [CENTAUR](https://arxiv.org/abs/2503.11650)

<h2 id="perception">Perception</h2>

<h3 id="08-occlusion-and-object-permanence">08 · Occlusion and object permanence</h3>

An agent has to keep track of what it can't currently see. [StEvo-Bench](https://eccv.ecva.net/virtual/2026/poster/4982) tests whether video world models evolve the state of the world independently of what's in view — insert an occluder or move the camera — and finds most of them let hidden objects disappear, a problem for any agent that plans with one. [BeyondSight](https://arxiv.org/abs/2607.09138) tackles the same principle in end-to-end driving: persistent actor hypotheses that survive occlusion, with nuScenes-Permanence for training and evaluation. Two other driving papers attack occlusion head-on: a worst-case search over hidden agents that stays consistent with what was seen before, and [localizing out-of-sight pedestrians through radar and camera reflections](https://eccv.ecva.net/virtual/2026/poster/4784). The same issue shows up in supervision: LEAD's privileged experts react to occluded actors the student can't see.

{% include figure.liquid path="assets/img/blog/eccv2026/fig04.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Object permanence: a car hidden behind a truck still exists. Many video world models drop it once it's out of view." caption="Object permanence: a car hidden behind a truck still exists. Many video world models drop it once it's out of view." %}

*More to read:* [RePL](https://eccv.ecva.net/virtual/2026/poster/3561)

<h3 id="09-open-world-perception">09 · Open-world perception</h3>

The other perception frontier is generalizing to what wasn't in the training set. [Fail2Drive](https://arxiv.org/abs/2604.08535) pairs every test route with an unseen twin — new objects, layouts and behaviors — and SOTA driving policies lose 22.8% success on average, sometimes ignoring objects clearly visible in the LiDAR. The fixes on show lean on foundation models: [BEVOpen3D](https://eccv.ecva.net/virtual/2026/poster/3368) distills a 2D vision-language model into an open-world LiDAR detector, [open-vocabulary BEV segmentation](https://eccv.ecva.net/virtual/2026/poster/3266) adds 3D geometric constraints, and [NegAS](https://eccv.ecva.net/virtual/2026/poster/3940) cuts out-of-distribution false positives by 25% on OpenImages. TokenGraph3D shows a frozen self-supervised point encoder already separates object instances without any class labels.

*More to read:* [real-time OOD detection](https://eccv.ecva.net/virtual/2026/poster/4106)

<h3 id="10-predict-fast-then-refine">10 · Predict fast, then refine</h3>

Reconstruction has three regimes — feed-forward (fast, few views), batch optimization (accurate, runs out of memory) and recurrent (scales, forgets) — and the field is converging on hybrids. [ReSplat](https://arxiv.org/abs/2510.08575) refines a feed-forward result with a learned render–measure–correct loop, [Learn2Splat](https://arxiv.org/abs/2605.15760) learns the optimizer, and [TTT3R](https://arxiv.org/abs/2509.26645) does streaming reconstruction as test-time training (2× better pose, 20 FPS, 6 GB, thousands of frames) — exactly what a robot or car needs for long-horizon scene memory.

{% include figure.liquid path="assets/img/blog/eccv2026/fig05.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Three reconstruction regimes, each with its own failure mode, and the hybrids that combine them: start feed-forward or recurrent, then refine at test time." caption="Three reconstruction regimes, each with its own failure mode, and the hybrids that combine them: start feed-forward or recurrent, then refine at test time." %}

*More to read:* [Flow4R](https://eccv.ecva.net/virtual/2026/poster/5342) · [Is Single-View Mesh Reconstruction Ready for Robotics?](https://arxiv.org/abs/2505.17966)

<h3 id="11-which-latent-space">11 · Which latent space?</h3>

Most video world models generate in a VAE latent, and that choice got attacked from both sides. LeCun: predict in an abstract space (JEPA) and ignore unpredictable detail. Andreas Geiger: predict actual geometry, not a proxy — [PointDiT](https://arxiv.org/abs/2607.02515) diffuses directly on point maps and keeps the thin structures and boundaries regression blurs. No consensus yet, but for anything that has to avoid collisions, Geiger's argument bites.

*More to read:* [Thermo-JEPA](https://eccv.ecva.net/virtual/2026/poster/3162)

<h3 id="12-perception-foundation-models">12 · Perception foundation models</h3>

Two kinds of pretrained model are becoming perception backbones.

**Reconstruction models.** [Map-Det3D](https://eccv.ecva.net/virtual/2026/poster/3781) detects objects directly in the metric 3D space that a feed-forward reconstructor builds from video. [MV2GF](https://eccv.ecva.net/virtual/2026/spotlight/6200) replaces ground-plane projection with Depth Anything 3 pointmaps. That swap alone adds **+7.9 MODA**, and with only two cameras it scores **57.6 vs 34.0**.

**Video generators.** [GenCeption](https://genception.github.io/) fine-tunes a text-to-video diffusion model into a single perception model. It matches or beats Depth Anything 3 and SAM3 on depth, normals, pose, and segmentation with **7–500×** less data. [ICDepth](https://arxiv.org/abs/2607.01677) and [GeoNeXt](https://arxiv.org/abs/2608.28549) do the same for video depth, from **0.8M** frames and **59K** samples respectively.

Both routes win on data efficiency and generalization more than on raw accuracy. Both also scale on supervision that's close to free: structure-from-motion, synthetic renders, LiDAR logs, or just video. The gap is that almost none of this has reached multi-camera 3D detection at driving scale yet — though Map-Det3D and GeoUP are early signs the reconstruction path is closing in.

<h2 id="simulation">Simulation</h2>

<h3 id="13-physics-reconstruction-generative">13 · Physics → reconstruction → generative</h3>

Geiger's framing: real data teaches a model how to act; simulation teaches it not to crash. Simulators went from physics engines (CARLA, and their robotics equivalents) to neural reconstruction (3DGS digital twins like [HUGSIM](https://arxiv.org/abs/2412.01718)) to generative models that reach beyond collected data. Today world models are used mostly for policy evaluation, data generation, and teachers for RL fine-tuning. World model usage as an on-robot policy via model-predictive control is too slow and less mature.

*More to read:* [SPEAR simulator](https://eccv.ecva.net/virtual/2026/poster/3273) · [Waymo World Model](https://waymo.com/blog/2026/02/the-waymo-world-model-a-new-frontier-for-autonomous-driving-simulation/)

<h3 id="14-fidelity-vs-reactivity">14 · Fidelity vs reactivity</h3>

No simulator has both yet. Log replay and 3DGS twins are faithful but can't react; box-world sims react but look nothing like sensors; generative models react and are expressive but drift. [CausalDrive](https://eccv.ecva.net/virtual/2026/poster/3753) is the current bar on reactivity (12.4 FPS, traffic that yields when prompted), and hybrids — 3DGS backgrounds plus diffusion, or [AlpaSim](https://huggingface.co/spaces/nvidia/AlpasimE2EClosedLoopChallenge2026) running on reconstruction or generation — are heading for the corner. Benchmarks follow the same path: NAVSIM → NAVSIM v2 → the AlpaSim closed-loop challenge.

{% include figure.liquid path="assets/img/blog/eccv2026/fig06.png" class="img-fluid rounded z-depth-1" zoomable=true alt="No simulator has both yet: replay and 3DGS twins are faithful but can't react, box-world sims react but have no realism, and hybrids and new benchmarks are moving toward the corner (schematic)." caption="No simulator has both yet: replay and 3DGS twins are faithful but can't react, box-world sims react but have no realism, and hybrids and new benchmarks are moving toward the corner (schematic)." %}

*More to read:* [LiSTAR](https://eccv.ecva.net/virtual/2026/poster/5672) · [Seoul World Model](https://eccv.ecva.net/virtual/2026/poster/4936)

<h3 id="15-conditioning-is-control">15 · Conditioning is control</h3>

A generator is only as controllable as the scene representation you condition it on. [PrITTI](https://arxiv.org/abs/2506.19117) conditions on primitives (ground plus cuboids) instead of voxels, and [Trajectory Forcing](https://eccv.ecva.net/virtual/2026/poster/5786) makes diffusion go layout → object → detail so every stage is editable. The consequence: a perception system's output format becomes the simulator's input format.

*More to read:* [Cosmos-Transfer1](https://arxiv.org/abs/2503.14492)

<h3 id="16-manufacturing-the-long-tail">16 · Manufacturing the long tail</h3>

Real logs contain almost none of the scenarios that matter, so they have to be manufactured. [World Engine](https://opendrivelab.com/WorldEngine/) finds real failures, rebuilds them in 3DGS, generates harder variants and RL-post-trains on them: **45.5% fewer collisions**, ~200 km on-road with zero disengagements. Adversarial generators are getting sharper too ([CompoSIA](https://eccv.ecva.net/virtual/2026/poster/3714) raises planner collisions 173%).

{% include figure.liquid path="assets/img/blog/eccv2026/fig07.png" class="img-fluid rounded z-depth-1" zoomable=true alt="The World Engine loop: find where the deployed model fails, rebuild those scenes, generate harder variants of them, and post-train on those variants." caption="The World Engine loop: find where the deployed model fails, rebuild those scenes, generate harder variants of them, and post-train on those variants." %}

*More to read:* [ReinDriveGen](https://eccv.ecva.net/virtual/2026/poster/3685)

<h3 id="17-on-vs-off-policy-generation">17 · On- vs off-policy generation</h3>

Generative simulators are trained on data they didn't produce — expert actions and clean context — then driven by a learner's actions and their own outputs. Expert-only models (GenAD, Vista) break on non-expert actions; [ReSim](https://arxiv.org/abs/2506.09981) fixes that by mixing in simulated bad driving. Rollouts drift unless trained on imperfect histories (CausalDrive, DAgger, diffusion forcing). The same gap applies to any robot world model trained on demonstrations: real data for grounding, sim data for coverage.

{% include figure.liquid path="assets/img/blog/eccv2026/fig08.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Generative simulators are trained on expert actions and clean context but used with a learner's actions and their own outputs; both gaps have known fixes." caption="Generative simulators are trained on expert actions and clean context but used with a learner's actions and their own outputs; both gaps have known fixes." %}

*More to read:* [OmniDreams](https://arxiv.org/abs/2606.03159) · [VATIX scaling laws](https://github.com/valeoai/VATIX)

<h2 id="data-engines">Data engines</h2>

<h3 id="18-coverage-not-frequency">18 · Coverage, not frequency</h3>

[TTCov](https://arxiv.org/abs/2607.22697) makes the concept space explicit — an LLM-built atlas of concepts — and picks training data to match how often each shows up in deployment. That's useful for a new environment, but matching frequency underweights rare, severe cases by design. Keep the atlas, change the target: a floor for every concept, weighted by severity. [Semantic-Drive](https://arxiv.org/html/2512.12012v2) is a good tool for filling it (0.966 vs 0.475 recall over CLIP for long-tail mining).

*More to read:* OpenDV-2K

<h3 id="19-get-ahead-of-the-long-tail">19 · Get ahead of the long tail</h3>

Dragomir Anguelov's line stuck with me: *"yesterday's long tail is today's torso."* The tail moves as capability improves, so chasing it after failures surface in deployment is a losing game. José Alvarez argued for test-driven development instead of reactive patching, and [Fail2Drive](https://arxiv.org/abs/2604.08535) gives the tool: paired tests of what a model has never seen (a 22.8% average success drop for SOTA).

{% include figure.liquid path="assets/img/blog/eccv2026/fig09.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Reactive: wait for a failure on the road, then patch. Proactive: list the scenario space, generate and test it in simulation, and fix failures before deployment." caption="Reactive: wait for a failure on the road, then patch. Proactive: list the scenario space, generate and test it in simulation, and fix failures before deployment." %}

*More to read:* [123D](https://arxiv.org/abs/2605.08084) · [the common task framework argument](https://kesai.eu/blog/2026-09-08-common-task-framework/)

<h3 id="20-automating-the-data-engine">20 · Automating the data engine</h3>

Every stage now has an automated tool — enumerate (TTCov), mine (Semantic-Drive), generate (CompoSIA, ReinDriveGen), auto-label (TokenGraph3D, RePL), train (World Engine) and validate (Fail2Drive, AlpaSim). What's missing is the map that connects them: a shared taxonomy of the long tail to enumerate against — in driving and, even more so, in open-world robotics.

*More to read:* KITTI-360 · [CooperScene](https://eccv.ecva.net/virtual/2026/poster/4574)

<h2 id="what-i-m-watching-next">What I'm watching next</h2>

A clean ablation that isolates what the world-model or language component actually buys a policy, in robotics or driving; world-action models that handle more than cameras — depth, LiDAR, force — at control rates; real-world head-to-head evaluations like RoboArena and the [AlpaSim closed-loop challenge](https://huggingface.co/spaces/nvidia/AlpasimE2EClosedLoopChallenge2026) (closing October 31); and a shared taxonomy of the long tail.

If you only read three things: [CausalDrive](https://eccv.ecva.net/virtual/2026/poster/3753), [World Engine](https://opendrivelab.com/WorldEngine/) and [Do World Action Models Generalize Better than VLAs?](https://arxiv.org/abs/2603.22078).

Workshops worth following: [DriveX](https://drivex-workshop.github.io/eccv2026/), [World Models in the Loop](https://eccv26woop.github.io/), [How to Build Effective World Models for Embodied AI](https://eccv26wmeai.github.io/), [Safe World Models](https://trustworthy-world-models.github.io/ECCV2026/). Numbers are as reported by the authors; closed-loop scores depend on the simulator and aren't comparable across them.
