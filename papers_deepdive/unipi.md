# UniPi — plan in video, then translate generated frames into robot actions

**Source:** [paper](https://arxiv.org/abs/2302.00111) · MIT / Google DeepMind / UC Berkeley / Georgia Tech / University of Alberta · NeurIPS 2023 · [Project](https://universal-policy.github.io/).
**Code:** not stated · license not stated · weights not stated.

## Motivations

Different environments have incompatible state and action spaces, which "hampers knowledge sharing, learning, and generalization" (§1). UniPi replaces those interfaces with images for behavior and text for goals. This also lets policy learning use internet video, since conventional world-model data must follow a "strict state-action-reward format" (§5).

## Methodology

UniPi defines a Unified Predictive Decision Process (UPDP) with a first-frame- and text-conditioned video planner

$$
\rho(\tau \mid x_0,c), \qquad \tau=[x_1,\ldots,x_H],
$$

followed by a task-specific policy $\pi(a_{0:H-1}\mid x_{0:H},c)$. Its implementation has two separately trained parts:

1. **Video planner.** A 1.7B-parameter Video U-Net is explicitly trained for "first frame conditioning during training" (§A.1). Tiling $x_0$ across denoising steps maintains scene consistency; coarse video generation followed by temporal super-resolution supplies hierarchical plans (§3.1).
2. **Action extraction.** A separate convolutional inverse-dynamics model predicts "7-dimensional controls" from generated observations (§A.2). It is trained independently on action-labeled simulator data; only 20k of the 200k combinatorial videos provide its labels (§A.4).
3. **Execution.** The generated action sequence may be replanned with MPC or run open-loop; all reported experiments use an "open-loop controller" for efficiency (§3.2).
4. **Real-world transfer.** The planner is pretrained on 14M video-text pairs, 60M image-text pairs, and LAION-400M, then "finetuning on the train split" of 7.2k Bridge video-text pairs (§4.3).

## How it differs / Limitations

| comparison | what UniPi does | consequence |
|---|---|---|
| Direct action policies | Plans in a shared image space, then adapts through a task-specific IDM | Supports differing action spaces while keeping plans "naturally interpretable by humans" (§1) |
| State-action diffusion | Generates future image frames instead of diffusing robot controls | Can use "internet-scale video data" but pays pixel-generation cost (§4.3) |
| Standard text-to-video | Conditions every denoising step on the tiled first observation | Reduces frames that "deviate significantly" from the observed scene (§3.1) |
| **Stated limitation: speed** | Photorealistic diffusion can take "a minute to generate" a plan | Too slow for reactive control without distillation or faster samplers (§6) |
| **Stated limitation: partial observability** | Experiments are "generally fully observed" | The model may hallucinate objects or physically unfaithful motion (§6) |
| **Skeptical read** | Planner and IDM are both trained; Bridge transfer fine-tunes the planner | Does not test whether a frozen model's native representation already exposes action structure |

## Conclusions

UniPi establishes that text-conditioned video can serve as a planning interface and that generated frames can be grounded to continuous control by a small IDM. In simulation it substantially exceeds the listed BC, trajectory-transformer, and action-diffusion baselines; for Bridge video generation, internet pretraining raises surrogate predicted success from 72.6% to 77.1% (§4.1–§4.3).

## Ties to the project

`proof-of-premise` · `contrast-baseline` · `scope-boundary`. UniPi is the canonical **generate-video-then-act** baseline: it shows that action-relevant trajectories can be synthesized from video data, then grounded with inverse dynamics. It does not discover low-level action structure inside a frozen representation. The planner is explicitly trained or fine-tuned, and the action mapping is manufactured by a separately trained IDM. Therefore **no UniPi checkpoint described in the paper is a valid frozen, action-blind probing target**. A separately released pre-robot Video Diffusion Models checkpoint would be in scope only if it had no action interface and could be kept frozen; availability is not stated.

The sharp comparison is: UniPi asks whether generated pixels can be translated into actions; this project asks whether action structure can be read directly from a frozen model's internal representation, without training an action module. UniPi's IDM is also a required baseline: a latent-space method must beat frame-pair inverse dynamics to show value beyond post-hoc visual action regression.

## Citations

- **Policy-as-video:** "sequential decision making problem as a text-conditioned video generation problem"; control actions are "extracted from the generated video" — Abstract.
- **Planner/IDM split:** "a diffusion model for the universal video-based planner" and "a task-specific action generator" — §3.
- **Planner training:** "explicitly train a constrained video synthesis model" — §3.1; "train each of our video diffusion models for 2M steps" — §A.1.
- **Separate action module:** "train a small task-specific inverse-dynamics model" and its training "is independent from the planner" — §3.2.
- **Real-world adaptation:** "pretrain UniPi on the pretraining dataset followed by finetuning" — §4.3.
- **Combinatorial result:** novel Place 60.1%, novel Relation 46.1%, versus the strongest listed baselines at 13.2% and 9.6% respectively — Table 1.
- **Multi-environment result:** UniPi 51.6% / 75.5% / 45.7% versus strongest listed baselines 14.8% / 21.7% / 10.5% — Table 3.
- **Pretraining result:** surrogate success 72.6% without pretraining and 77.1% with pretraining — Table 4.
- **Limitations:** "can take a minute"; environments are "generally fully observed"; models may "make hallucination of objects or movements" — §6.

## References — pure-video & video-generation

- **Video Diffusion Models** (Ho et al., 2022) — UniPi's base Video U-Net architecture. **[target]** if an action-blind checkpoint is available; availability is not stated.
- **Imagen Video** (Ho et al., 2022) — text-to-video training data and architecture lineage used for internet pretraining. **[background]**
- **Phenaki** (Villegas et al., 2022) — open-domain text-to-video comparison. **[background]**
- **Diffuser** (Janner et al., 2022) — diffusion planning over state-action trajectories; UniPi's direct action-space foil. **[background]**
- **Video PreTraining (VPT)** (Baker et al., 2022) — labels internet video with a trained IDM, an earlier video-to-action route. **[background]**