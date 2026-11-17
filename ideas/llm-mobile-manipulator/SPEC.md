# Spec: LLM-driven wheeled mobile manipulator

## Concept

- **Hardware:** wheeled mobile base + multi-joint arm + gripper + cameras (RGB-D).
- **Skill library (pre-coded):** navigate_to(location), find(object), pick(object), place(object, location), open(drawer/door), push, hand_over. Each skill has a short natural-language description and a success-likelihood (affordance) estimate given the current state.
- **Decision maker:** an LLM or VLM takes the user's instruction plus a scene description (or images) and outputs a sequence of skills, choosing only from the allowed skill set.
- **Execution loop:** run a skill, check success with the vision model, replan on failure.
- **Safety layer:** hard constraints (speed limits, keep-out zones, force limits, human detection) that the LLM cannot override.

## Prior art: patents

| Number | Title | Notes |
|---|---|---|
| [US20230311335A1](https://patents.google.com/patent/US20230311335A1/en) | Natural language control of a robot (Google) | LLM processes a free-form instruction; skills have language descriptions and affordance functions giving probability of success from the current state. Basis of SayCan. |
| [WO2023192497A1](https://patents.google.com/patent/WO2023192497A1/en) | Natural language control of a robot | PCT counterpart of the above. |
| [JP7879257B2](https://patents.google.com/patent/JP7879257B2/en) | Natural language control of robots | Granted Japanese counterpart. |
| [US20240253224A1](https://patents.google.com/patent/US20240253224A1/en) | Robot systems, methods, control modules, and computer program products that leverage large language models | LLM returns a task plan made only of allowed instructions from an instruction set. Assignee not verified here. |
| [US20240359319A1](https://patents.google.com/patent/US20240359319A1/en) | Robot systems, methods, control modules, and computer program products that leverage large language models | Related filing. Assignee not verified here. |

Search further on Google Patents / USPTO with terms like "large language model", "mobile manipulator", "task plan", and CPC class B25J 9/16.

## Prior art: research

- **SayCan**, "Do As I Can, Not As I Say: Grounding Language in Robotic Affordances" ([arXiv 2204.01691](https://arxiv.org/pdf/2204.01691)). LLM (PaLM) + skill library + affordance scores, on an Everyday Robots wheeled mobile manipulator with a 7-DoF arm. Coverage: [Hackster](https://www.hackster.io/news/google-s-palm-saycan-gives-everyday-robots-a-better-understanding-of-natural-language-commands-da1221e2e36d), [InfoQ](https://www.infoq.com/news/2022/09/google-robot-saycan/), [Fast Company](https://www.fastcompany.com/90779334/google-robot-research-everyday-robots-palm-saycan), [Everyday Robots](https://www.everydayrobots.com/thinking/palm-saycan-googleresearch).
- **TidyBot** (IROS 2023), mobile manipulator that tidies rooms, LLM infers user placement preferences ([PDF](https://gfx.cs.princeton.edu/gfx/pubs/Wu_2023_TPR/tidybot_IROS23.pdf)).
- **LLM-GROP** (IJRR 2025), LLM common sense for object arrangement, joint navigation + manipulation planning ([UT Austin](https://www.cs.utexas.edu/~pstone/Papers/bib2html/b2hd-xiaohan_ijrr2025.html)).
- **CENTAURO loco-manipulation planning**, LLM builds task graphs from a behavior library, VLM checks success/failure ([arXiv 2408.08282](https://arxiv.org/html/2408.08282v1)).
- **ADM-Planner**, LLM long-horizon planning for mobile manipulators with dynamic memory ([arXiv 2609.29212](https://arxiv.org/pdf/2609.29212)).
- **Inria Tiago++ in euROBIN** coopetition, vocal instructions to household tasks ([arXiv 2412.17861](https://arxiv.org/html/2412.17861v1)).
- Grounding language with visual affordances ([arXiv 2210.01911](https://arxiv.org/pdf/2210.01911)); affordance-guided RL via visual prompting ([arXiv 2407.10341](https://arxiv.org/pdf/2407.10341)).

## Possible openings (where something new may still exist)

- Failure detection and recovery for grasps, with the LLM reasoning about why a skill failed.
- Verifiable safety: formally checking an LLM plan against constraints before execution.
- Learning new skills from one demonstration and auto-generating their descriptions and affordance models.
- Domain-specific products (hospital, lab, retail) where the value is integration and reliability, not the planner.

## Suggested stack

ROS 2, Nav2 (navigation), MoveIt 2 (arm motion), an open VLM for perception and planning, Isaac Sim or Gazebo for simulation.
