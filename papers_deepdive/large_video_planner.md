# Large Video Planner — train video generation at scale, then retarget generated human motion to robots

**Source:** [paper](https://arxiv.org/abs/2512.15840) · MIT / UC Berkeley / Harvard · May 8, 2026 · [Project](https://www.boyuan.space/large-video-planner/).
**Code:** released through the project page; repository URL not stated in the paper · license not stated · model, data, and training code released ("open-source our model, data, and training code," §6).

## Motivations

VLA transfer is asymmetric because web image-language pretraining is followed by a "narrow amount of robot data" (§1). LVP instead treats internet video as a source of state-action plans: videos "naturally encode state-action plans" and expose temporal dynamics absent from static image-text pairs (§1).

## Methodology

LVP is a two-stage generate-video-then-act system:

1. **Video planner.** Starting from Wan 2.1 I2V 14B, the authors "train on the full dataset for 60k steps" and then fine-tune 10k steps on low-camera-motion clips (§3.1). Diffusion Forcing assigns separate noise levels to history and future frames; History Guidance strengthens adherence to context images.
2. **Action-focused pretraining data.** LVP-1M contains "1.4M short clips" from four robot and four human-video sources, temporally aligned to human speed and paired with action-centric captions (§3.2).
3. **Action extraction.** HaMeR estimates per-frame MANO hands, MegaSAM supplies 4D-consistent depth and camera poses, and Dex-Retargeting maps hand keypoints to robot joints (§3.3). The wrist trajectory is calibrated into the robot frame, then cuRobo solves arm IK.
4. **Execution.** The resulting arm and hand trajectories are synchronized and executed open-loop; the paper states the "overall robot execution framework is run in an open-loop manner" (§5).

For flow-matching noise level $k$,

$$
z_k=(1-k)z_0+k\epsilon, \qquad \epsilon\sim\mathcal N(0,I),
$$

and the model learns the video-latent flow from noisy context (§3.1). This noise is a candidate analysis surface, but LVP itself reads actions from decoded pixels rather than from noise or denoising states.

## How it differs / Limitations

| vs prior practice | what LVP does | limitation or project consequence |
|---|---|---|
| VLA robot foundation models | Uses video as the "primary modality" and generates human/robot motion plans (§1) | It does not directly emit actions; control depends on post-hoc reconstruction and retargeting |
| Generic I2V models | Continues training Wan on action-focused video and uses Diffusion Forcing plus History Guidance | LVP's 59.3% task-complete rate exceeds Wan's 39.3% on the 100-task test (§4.2, Table 2), but this does not isolate generic pretraining from robot/action-video specialization |
| Learned inverse dynamics | Uses geometric reconstruction and embodiment retargeting rather than an IDM | Requires visible hands, metric reconstruction, camera-to-robot calibration, IK, and morphology-specific retargeting (§3.3) |
| **Stated limitation: latency** | A plan "takes several minutes" on one A100 (§5) | Direct real-time deployment is intractable |
| **Stated limitation: extraction errors** | Open-source 4D and hand estimators "can make mistakes" (§5) | Perception failures propagate directly to control |
| **Stated limitation: embodiment** | Parallel-jaw retargeting is "challenging due to its much lower" degrees of freedom (§5) | Human-hand plans do not define a universal robot action representation |
| **Stated limitation: feedback** | Execution is open-loop and "not sufficient for accomplishing dexterous tasks" (§5) | Reported success does not establish robust closed-loop control |

## Conclusions

On 100 independently proposed tasks, LVP reaches 59.3% average task-complete and 44.0% perfect-task success, versus 39.3% and 20.5% for Wan (§4.2, Table 2). Real-robot trials show nonzero zero-shot execution across parallel-gripper and dexterous-hand tasks, but success is task-dependent and often below 50% (§4.3, Figure 8).

## Ties to the project

`proof-of-premise` · `contrast-baseline` · `valid-probing-target` (qualified).

LVP is a strong proof that an action-blind video generator can synthesize task-relevant motion and that generated motion can be grounded to control. It is also the exact **generate-video-then-act** baseline the project aims to avoid: actions are extracted after pixel decoding through HaMeR, MegaSAM, Dex-Retargeting, calibration, and IK, rather than discovered inside the frozen representation.

The released LVP checkpoint has no action input or output, so it is technically an action-blind probing target. However, it was explicitly trained on robot demonstrations and action-focused human clips. A positive probe could therefore reflect **soft action leakage** from specialized data. The clean experiment is paired: probe base Wan 2.1 and frozen LVP with the same bottleneck. Improvement only in LVP would weaken the claim that action readability is a free property of generic internet-scale video pretraining; similar structure in base Wan would strengthen it.

LVP also provides a direct test surface for the new noise hypothesis: compare raw noise, intermediate denoising states, and local noise-to-motion Jacobians against its decoded-pixel retargeting pipeline. The latent method must improve label efficiency, robustness, or latency; merely matching the final video trajectory would not establish value.

## Citations

- **Video-first premise:** videos "capture spatio-temporal sequences of states and actions" — Abstract; "naturally encode state-action plans" — §1.
- **Generation and extraction:** produces "zero-shot video plans" that are post-processed into "executable robot actions" — Abstract.
- **Backbone and training:** "pre-trained video foundation model, WAN 2.1 14B"; "60k steps" plus "additional 10k steps" — §3.1.
- **Dataset:** "1.4M short clips" and "4.1 million captioned clips" after repeated captions — §3.2.
- **Retargeting stack:** "hand reconstruction model: HaMeR," "dynamic scene reconstruction model, MegaSAM," and "use Dex-Retargeting" — §3.3.
- **Video evaluation:** "100 in-the-wild manipulation prompts"; LVP 59.3% Level 3 and 44.0% Level 4 — §4.2, Table 2.
- **Robot evaluation:** two "distinct robot morphologies" and released-checkpoint π0/OpenVLA baselines without fine-tuning — §4.3.
- **Limitations:** "takes several minutes"; estimators "can make mistakes"; execution is "open-loop" — §5.
- **Release:** "open-source our model, data, and training code" — §6.

## References — pure-video & video-generation

- **Wan: Open and Advanced Large-Scale Video Generative Models** (Wan Team, 2025) — initialization and strongest ablation baseline; the base action-blind checkpoint remains the cleaner project target. **[target]**
- **Diffusion Forcing** (Chen et al., 2024) — independent per-frame noise levels for flexible history conditioning and rollout. **[mechanism]**
- **History-Guided Video Diffusion** (Song et al., 2025) — context-frame guidance used to improve temporal grounding. **[mechanism]**
- **Learning Universal Policies via Text-Guided Video Generation / UniPi** (Du et al., 2023) — earlier generate-video-then-IDM control pipeline. **[background]**
- **Gen2Act** (Bharadhwaj et al., 2024) — human video generation for generalizable robot manipulation. **[background]**
- **Cosmos World Foundation Model Platform** (Agarwal et al., 2025) — action-blind video baseline and alternative probing target. **[target]**

**Reference audit commands:** `grep -ic 'Wan' /tmp/video_planner_enables_control.txt` → 50; `grep -ic 'Diffusion forcing' ...` → 13; `grep -ic 'History-guided video diffusion' ...` → 1; `grep -ic 'Learning universal policies via text-guided video generation' ...` → 1; `grep -ic 'Gen2act' ...` → 1; `grep -ic 'Cosmos' ...` → 5; `grep -ic 'Flow matching' ...` → 3.