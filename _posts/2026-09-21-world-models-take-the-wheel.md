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
  - name: Policy
    subsections:
      - name: "01 · WAM vs VLA as policies"
      - name: "02 · Closed-loop policy learning"
      - name: "03 · Self-play overfits"
  - name: Perception
    subsections:
      - name: "04 · Geometry-first perception backbones"
      - name: "05 · Open-world perception"
      - name: "06 · Occlusion and object permanence"
  - name: Simulation
    subsections:
      - name: "07 · Manufacturing the long tail"
      - name: "08 · Fidelity vs reactivity"
      - name: "09 · Generative sim train/serve gap"
  - name: Data engines
    subsections:
      - name: "10 · Borrow experience from cheaper embodiments"
      - name: "11 · Data engines for the long tail"
  - name: "What I'm watching next"
---
{% include figure.liquid path="assets/img/blog/eccv2026/hero.jpg" class="img-fluid rounded z-depth-1" zoomable=true alt="Illustration of a world model: streams of real camera frames, LiDAR points and an action arrow flow into a glowing translucent lattice cube, inside which a city intersection with a vehicle is being generated; strips of possible future video frames fan out on the other side." caption="At ECCV 2026 world models were the protagonist — as a simulator, evaluator, and candidate policy for embodied AI." %}

Computer vision is turning its focus to physical AI. At ECCV 2026 all three keynotes were about models that act in the physical world rather than just look at it: Grauman on 4D egocentric understanding, LeCun on action-conditioned world models for planning, Shotton on learning over engineering in a decade of driving. The idea underneath much of physical AI discussion was the world model that predicts what happens next and increasingly reasons about what to do about it. At least six workshops were built around world models, and the same backbones kept showing up in robot manipulation, egocentric video, and autonomous driving.

My own lens is driving, which Jamie Shotton called the first proving ground for embodied AI, but most of what follows transfers to robotics. Here are eleven takeaways, grouped into four areas.

<h2 id="the-program-at-a-glance">The program at a glance</h2>

**10,473** submissions · **2,883** accepted (27.5%) · **86** workshops

The building blocks of physical AI dominate the program: image and video generation plus vision-language models account for roughly two in five papers, and 3D is the largest single subfield and every main-track award went to 3D or geometry work. Robotics and embodied AI (~6.5%) and driving (~6%) are small by paper count, but they owned the stage: all the keynotes, and many of the largest workshops.

{% include figure.liquid path="assets/img/blog/eccv2026/fig01.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Bar chart of the ten largest ECCV 2026 subfields, with embodied AI and driving highlighted." caption="Ten largest subfields — share of 1,131 categorized papers." %}

Highlighted: embodied AI and driving subfields.

Shares from a third-party categorization of 1,131 of the 2,883 accepted papers ([papernotes](https://en.papernotes.org/ECCV2026/)).

<h2 id="first-what-s-a-world-model">First, what's a world model?</h2>

The term *world model* gets slapped on many different things, and the conversation gets muddled when they're treated as the same thing. One clean taxonomy to tell different models apart is the input-output structure. The same split applies whether the agent is a robot arm or a car.

{% include figure.liquid path="assets/img/blog/eccv2026/fig02.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Three stacks in physical AI — video generator, action-conditioned world model, and world-action model — told apart by where the action sits: absent, an input, or an output. The left column is text/image-to-video without actions; only the middle column is a transition model for planning in the usual sense. Cosmos 3 appears in all three because one foundation supports each mode. A VLA has the same inputs and outputs as a world-action model, but a vision-language backbone instead of a video one." caption="Three stacks in physical AI — video generator, action-conditioned world model, and world-action model — told apart by where the action sits: absent, an input, or an output. The left column is text/image-to-video without actions; only the middle column is a transition model for planning in the usual sense. Cosmos 3 appears in all three because one foundation supports each mode. A VLA has the same inputs and outputs as a world-action model, but a vision-language backbone instead of a video one." %}

Most of the pixel-generating models in all three columns share a latent diffusion/flow transformer backbone, and unified foundation models like [Cosmos 3](https://developer.nvidia.com/blog/develop-physical-ai-reasoning-world-and-action-models-with-nvidia-cosmos-3/) now support each mode. The left column is generative video in the usual sense — pretraining and synthetic data — not an agent's transition model until actions are wired in. One real architectural split among the action-aware stacks is generative versus joint-embedding (JEPA-style) models, which predict latent states instead of pixels.

ECCV also split on WAM (video backbone) versus VLA (language backbone), with hybrids such as [StructVLA](https://eccv.ecva.net/virtual/2026/poster/3187). The same families appear in manipulation, driving, and navigation — as policies, simulators, or teachers — and fair cross-domain comparisons are still rare. The takeaways, trends, and challenges below follow four key topical areas: robot policy, perception, simulation, and data engines. Driving is the running example, but most of it transfers to robotics.

<h2 id="policy">Policy</h2>

<h3 id="01-wam-vs-vla-as-policies">01 · WAM vs VLA as policies</h3>

For robot policy applications ECCV had plenty of WAM and VLA-based works on display, but direct comparisons were rare and often unfairly matched. In robotics, [a robustness study](https://arxiv.org/abs/2603.22078) finds WAMs ahead on RoboTwin-Plus (74.2% vs 58.6% for π0.5) but behind on LIBERO-Plus (82.2% vs 85.7%), and [NVIDIA reports](https://developer.nvidia.com/blog/pretrained-to-imagine-fine-tuned-to-act-the-rise-of-world-action-models/) DreamZero leading RoboArena. In driving, [OmniDreams](https://arxiv.org/abs/2606.03159) has a WAM beating VLA-based Alpamayo 1.5 at ~1/5 the compute. Indoor navigation in Habitat mostly repeated the same split — navigation world models versus instruction-following VLAs — while open-world outdoor nav stayed thin.

[PRISM](https://arxiv.org/abs/2608.01201) finds VLM supervision helps driving through the latents it shapes, not through reasoning, and [Scaling Verification](https://eccv.ecva.net/virtual/2026/poster/5640) (Finn, Pavone) shows a test-time verifier beating the same data spent on more VLA pre-training. I would not bet on WAM vs VLA as the decisive fork: with enough data and compute these are all large multimodal transformers, and the pretraining objective (language and VQA vs video and scene dynamics) may matter more than the branding. What repeatedly showed up instead was post-training and evaluation — closed-loop RL or distillation, verifiers and scorers at test time, and benchmarks that stress deployment rather than open-loop imitation.

*More to read:* [Sim-to-real in dexterous VLAs](https://eccv.ecva.net/virtual/2026/poster/5392)

<h3 id="02-closed-loop-policy-learning">02 · Closed-loop policy learning</h3>

Wayve, NVIDIA, Waymo, Zenseact and KE:SAI all gave the same diagnosis: behavior cloning learns *predictions*, not *behaviors*. Bernhard Jaeger's talk laid out the two recipes that work: RL post-training on top of an imitation policy (what ships today), and a privileged RL teacher distilled on-policy into a sensor-based student (the research frontier). The same split is playing out in robot learning. NVIDIA's [survey of closed-loop training](https://research.nvidia.com/labs/avg/publication/karkus.igl.etal.pami2025/) is the best overview.

[LEAD](https://arxiv.org/abs/2512.20563) traced the CARLA plateau to what the privileged expert sees that the student can't: occluded actors, noiseless state, and a richer route. Removing those gaps gave **+8 driving score** on Bench2Drive. Anyone distilling from a privileged teacher — in sim-to-real robotics as much as driving — has the same exposure, and so does perception ([+44% long-range BEV](https://eccv.ecva.net/virtual/2026/poster/5994) from an overhead teacher).

{% include figure.liquid path="assets/img/blog/eccv2026/fig03.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Whatever the teacher sees that the student can't (occluded actors, clean state, a richer route) ends up in the target the student is asked to reproduce." caption="Whatever the teacher sees that the student can't (occluded actors, clean state, a richer route) ends up in the target the student is asked to reproduce." %}

*More to read:* [BeyondDrive](https://eccv.ecva.net/virtual/2026/poster/3394) · [CaRL](https://arxiv.org/abs/2504.17838v3) · [UniDrive-WM](https://arxiv.org/abs/2601.04453) · [WTA→GMM forecasting](https://eccv.ecva.net/virtual/2026/poster/4060) · [SparseDriveV2](https://eccv.ecva.net/virtual/2026/poster/3597)

<h3 id="03-self-play-overfits">03 · Self-play overfits</h3>

Self-play scales beautifully — [TerraZero](https://arxiv.org/abs/2607.13028) runs 1.3M agent-steps/s and leads the InterPlan long-tail benchmark with no human demonstrations — but [BehaviorBench](https://arxiv.org/abs/2605.10034) shows pure self-play policies overfit to their own traffic and crash more against other agent models. Evaluate against a panel of behaviors, not one.

*More to read:* [ZTRS](https://eccv.ecva.net/virtual/2026/poster/4135) · [Scaling Self-Play](https://arxiv.org/html/2606.19641) · [CENTAUR](https://arxiv.org/abs/2503.11650)

<h2 id="perception">Perception</h2>

<h3 id="04-geometry-first-perception-backbones">04 · Geometry-first perception backbones</h3>

Two kinds of pretrained model are becoming perception backbones, both betting on geometry over raw semantics.

**Reconstruction models.** [Map-Det3D](https://eccv.ecva.net/virtual/2026/poster/3781) detects objects directly in the metric 3D space that a feed-forward reconstructor builds from video. [MV2GF](https://eccv.ecva.net/virtual/2026/spotlight/6200) replaces ground-plane projection with Depth Anything 3 pointmaps — **+7.9 MODA**, and with only two cameras **57.6 vs 34.0**. [GeoUP](https://arxiv.org/abs/2608.13147) adapts a VGGT-style latent to calibrated streaming multi-camera driving for unified depth, detection, and occupancy. Hybrid scene memory follows the same pattern: feed-forward or recurrent reconstruction, then refine at test time ([ReSplat](https://arxiv.org/abs/2510.08575), [TTT3R](https://arxiv.org/abs/2509.26645)).

{% include figure.liquid path="assets/img/blog/eccv2026/fig05.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Three reconstruction regimes, each with its own failure mode, and the hybrids that combine them: start feed-forward or recurrent, then refine at test time." caption="Three reconstruction regimes, each with its own failure mode, and the hybrids that combine them: start feed-forward or recurrent, then refine at test time." %}

**Video generators.** [GenCeption](https://genception.github.io/) fine-tunes text-to-video diffusion into a single perception model, matching or beating Depth Anything 3 and SAM3 on depth, normals, pose, and segmentation with **7–500×** less data. [ICDepth](https://arxiv.org/abs/2607.01677) and [GeoNeXt](https://arxiv.org/abs/2608.28549) do the same for video depth from **0.8M** frames and **59K** samples. [RoMan-4D](https://eccv.ecva.net/virtual/2026/poster/3281) and [VGGT-World](https://eccv.ecva.net/virtual/2026/poster/3788) push geometry-grounded video world models for manipulation.

Both routes have different strengths and aim to generalize on more than on peak leaderboard scores, using cheap supervision: SfM, synthetic renders, LiDAR logs, or video. Representations still differ — generative VAE latents, JEPA-style abstract states, explicit geometry ([PointDiT](https://arxiv.org/abs/2607.02515) on point maps); [RAE-NWM](https://arxiv.org/abs/2603.09241) predicts navigation rollouts in dense DINOv2 tokens (~79% on Habitat image-goal). For operating in the real world, explicit geometry still looks like the safer bet than pixels alone.

*More to read:* [PhysMani](https://eccv.ecva.net/virtual/2026/poster/4493) · [What if? (world models for spatial reasoning)](https://eccv.ecva.net/virtual/2026/poster/4721) · [Flow4R](https://eccv.ecva.net/virtual/2026/poster/5342) · [Thermo-JEPA](https://eccv.ecva.net/virtual/2026/poster/3162)

<h3 id="05-open-world-perception">05 · Open-world perception</h3>

Generalizing to what wasn't in the training set is the other big frontier. [Fail2Drive](https://arxiv.org/abs/2604.08535) pairs every test route with an unseen twin — new objects, layouts and behaviors — and SOTA driving policies lose 22.8% success on average, sometimes ignoring objects clearly visible in the LiDAR. [360CityArena](https://arxiv.org/abs/2608.08814) is the navigation counterpart: real Akihabara panoramas in a photorealistic city benchmark, where Gemini 2.5 Flash reaches 17.1% next to 77.3% for humans. The fixes on show lean on foundation models: [BEVOpen3D](https://eccv.ecva.net/virtual/2026/poster/3368) distills a 2D vision-language model into an open-world LiDAR detector, [open-vocabulary BEV segmentation](https://eccv.ecva.net/virtual/2026/poster/3266) adds 3D geometric constraints, and [NegAS](https://eccv.ecva.net/virtual/2026/poster/3940) cuts out-of-distribution false positives by 25% on OpenImages. TokenGraph3D shows a frozen self-supervised point encoder already separates object instances without any class labels.

*More to read:* [real-time OOD detection](https://eccv.ecva.net/virtual/2026/poster/4106) · [Is Single-View Mesh Reconstruction Ready for Robotics?](https://arxiv.org/abs/2505.17966)

<h3 id="06-occlusion-and-object-permanence">06 · Occlusion and object permanence</h3>

An agent has to keep track of what it can't currently see. [StEvo-Bench](https://eccv.ecva.net/virtual/2026/poster/4982) tests whether action-conditioned generative video models evolve the state of the world independently of what's in view — insert an occluder or move the camera — and finds most of them let hidden objects disappear, a problem for any agent that plans with one. [BeyondSight](https://arxiv.org/abs/2607.09138) tackles the same principle in end-to-end driving: persistent actor hypotheses that survive occlusion, with nuScenes-Permanence for training and evaluation. Two other driving papers attack occlusion head-on: a worst-case search over hidden agents that stays consistent with what was seen before, and [localizing out-of-sight pedestrians through radar and camera reflections](https://eccv.ecva.net/virtual/2026/poster/4784).

{% include figure.liquid path="assets/img/blog/eccv2026/fig04.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Object permanence: a car hidden behind a truck still exists. Many generative video models drop it once it's out of view." caption="Object permanence: a car hidden behind a truck still exists. Many generative video models drop it once it's out of view." %}

*More to read:* [RePL](https://eccv.ecva.net/virtual/2026/poster/3561)

<h2 id="simulation">Simulation</h2>

<h3 id="07-manufacturing-the-long-tail">07 · Manufacturing the long tail</h3>

Real logs contain almost none of the scenarios that matter, so they have to be manufactured. [World Engine](https://opendrivelab.com/WorldEngine/) finds real failures, rebuilds them in 3DGS, generates harder variants and RL-post-trains on them: **45.5% fewer collisions**, ~200 km on-road with zero disengagements. Adversarial generators are getting sharper too ([CompoSIA](https://eccv.ecva.net/virtual/2026/poster/3714) raises planner collisions 173%).

{% include figure.liquid path="assets/img/blog/eccv2026/fig07.png" class="img-fluid rounded z-depth-1" zoomable=true alt="The World Engine loop: find where the deployed model fails, rebuild those scenes, generate harder variants of them, and post-train on those variants." caption="The World Engine loop: find where the deployed model fails, rebuild those scenes, generate harder variants of them, and post-train on those variants." %}

*More to read:* [ReinDriveGen](https://eccv.ecva.net/virtual/2026/poster/3685)

<h3 id="08-fidelity-vs-reactivity">08 · Fidelity vs reactivity</h3>

Simulators moved from physics engines (CARLA and robotics equivalents) to neural reconstruction (3DGS twins like [HUGSIM](https://arxiv.org/abs/2412.01718)) to generative models that reach beyond collected data. Today world models are used mostly for policy evaluation, data generation, and RL teachers — not yet as on-robot MPC policies.

No simulator has both fidelity and reactivity yet. Log replay and 3DGS twins are faithful but can't react; box-world sims react but look nothing like sensors; generative models react and are expressive but drift. [CausalDrive](https://eccv.ecva.net/virtual/2026/poster/3753) is the current bar on reactivity (12.4 FPS, traffic that yields when prompted), and hybrids — 3DGS backgrounds plus diffusion, or [AlpaSim](https://huggingface.co/spaces/nvidia/AlpasimE2EClosedLoopChallenge2026) on reconstruction or generation — are heading for the corner. A generator is only as controllable as what you condition on: [PrITTI](https://arxiv.org/abs/2506.19117) uses primitives instead of voxels, and [Trajectory Forcing](https://eccv.ecva.net/virtual/2026/poster/5786) makes diffusion go layout → object → detail. Benchmarks follow the same arc: NAVSIM → NAVSIM v2 → AlpaSim.

{% include figure.liquid path="assets/img/blog/eccv2026/fig06.png" class="img-fluid rounded z-depth-1" zoomable=true alt="No simulator has both yet: replay and 3DGS twins are faithful but can't react, box-world sims react but have no realism, and hybrids and new benchmarks are moving toward the corner (schematic)." caption="No simulator has both yet: replay and 3DGS twins are faithful but can't react, box-world sims react but have no realism, and hybrids and new benchmarks are moving toward the corner (schematic)." %}

*More to read:* [SPEAR simulator](https://eccv.ecva.net/virtual/2026/poster/3273) · [Waymo World Model](https://waymo.com/blog/2026/02/the-waymo-world-model-a-new-frontier-for-autonomous-driving-simulation/) · [LiSTAR](https://eccv.ecva.net/virtual/2026/poster/5672) · [Cosmos-Transfer1](https://arxiv.org/abs/2503.14492)

<h3 id="09-generative-sim-train-serve-gap">09 · Generative sim train/serve gap</h3>

Generative simulators are trained on expert actions and clean context, then driven by a learner's actions and their own outputs. Expert-only models (GenAD, Vista) break on non-expert actions; [ReSim](https://arxiv.org/abs/2506.09981) fixes that by mixing in simulated bad driving. Rollouts drift unless trained on imperfect histories (CausalDrive, DAgger, diffusion forcing). The same gap applies to robot world models trained on demonstrations: real data for grounding, sim data for coverage.

{% include figure.liquid path="assets/img/blog/eccv2026/fig08.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Generative simulators are trained on expert actions and clean context but used with a learner's actions and their own outputs; both gaps have known fixes." caption="Generative simulators are trained on expert actions and clean context but used with a learner's actions and their own outputs; both gaps have known fixes." %}

*More to read:* [OmniDreams](https://arxiv.org/abs/2606.03159) · [VATIX scaling laws](https://github.com/valeoai/VATIX)

<h2 id="data-engines">Data engines</h2>

<h3 id="10-borrow-experience-from-cheaper-embodiments">10 · Borrow experience from cheaper embodiments</h3>

Robot and vehicle data is expensive; video of people and dashcams is not. The trick that kept recurring is a structural bridge between the two. Grauman's EgoExo-WM trains egocentric world models from third-person video using body pose as the bridge. [DexWM](https://eccv.ecva.net/virtual/2026/poster/5625) learns dexterous hand–object world models from human video and transfers zero-shot to a robot hand, beating Diffusion Policy by over 50%. In driving, Waymo's [Sensor2Sensor](https://arxiv.org/html/2605.22809) turns dashcam video into multi-camera + LiDAR logs for a target vehicle, and [roadside sensors can act as teachers](https://eccv.ecva.net/virtual/2026/poster/4075). Yunzhu Li made the general case: structured world models as scalable data engines for robotics.

*More to read:* [One Demonstration Is Enough for Real-World Robotic RL](https://eccv.ecva.net/virtual/2026/poster/5855)

<h3 id="11-data-engines-for-the-long-tail">11 · Data engines for the long tail</h3>

Dragomir Anguelov's line stuck with me: *"yesterday's long tail is today's torso."* The tail moves as capability improves, so chasing failures only after deployment is a losing game. José Alvarez argued for test-driven development instead of reactive patching, and [Fail2Drive](https://arxiv.org/abs/2604.08535) gives the tool: paired tests of what a model has never seen (a 22.8% average success drop for SOTA).

{% include figure.liquid path="assets/img/blog/eccv2026/fig09.png" class="img-fluid rounded z-depth-1" zoomable=true alt="Reactive: wait for a failure on the road, then patch. Proactive: list the scenario space, generate and test it in simulation, and fix failures before deployment." caption="Reactive: wait for a failure on the road, then patch. Proactive: list the scenario space, generate and test it in simulation, and fix failures before deployment." %}

[TTCov](https://arxiv.org/abs/2607.22697) makes the concept space explicit — an LLM-built atlas of concepts — and picks training data to match deployment frequency. That underweights rare, severe cases by design; keep the atlas, change the target to a floor per concept weighted by severity. [Semantic-Drive](https://arxiv.org/html/2512.12012v2) helps fill it (0.966 vs 0.475 recall over CLIP for long-tail mining).

Every stage now has an automated tool — enumerate (TTCov), mine (Semantic-Drive), generate (CompoSIA, ReinDriveGen), auto-label (TokenGraph3D, RePL), train (World Engine) and validate (Fail2Drive, AlpaSim). Less clear is a shared taxonomy of rare scenarios that every group could use to drive the pipeline — in driving and, even more, in open-world robotics.

*More to read:* [123D](https://arxiv.org/abs/2605.08084) · [the common task framework argument](https://kesai.eu/blog/2026-09-08-common-task-framework/) · [CooperScene](https://eccv.ecva.net/virtual/2026/poster/4574)

<h2 id="what-i-m-watching-next">What I'm watching next</h2>

- A controlled ablation that separates what a VLM vs WAM actually buys for robotics policy when both are looking increasingly similar.
- VLA and WAMs that are multi-modal and can take multi-camera, depth, LiDAR, RaDAR, or force at control rates.
- Real closed-loop evaluations like RoboArena and the [AlpaSim closed-loop challenge](https://huggingface.co/spaces/nvidia/AlpasimE2EClosedLoopChallenge2026) (closing October 31). 
- A broader recipe or approach to taxonomy of rare scenarios for data engines.

If you only read three things: [CausalDrive](https://eccv.ecva.net/virtual/2026/poster/3753), [World Engine](https://opendrivelab.com/WorldEngine/) and [Do World Action Models Generalize Better than VLAs?](https://arxiv.org/abs/2603.22078).

Workshops worth following: [DriveX](https://drivex-workshop.github.io/eccv2026/), [World Models in the Loop](https://eccv26woop.github.io/), [How to Build Effective World Models for Embodied AI](https://eccv26wmeai.github.io/), [Safe World Models](https://trustworthy-world-models.github.io/ECCV2026/).
