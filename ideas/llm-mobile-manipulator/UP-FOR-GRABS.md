# Up for grabs: LLM-driven wheeled mobile manipulator

I thought about this but stopped for lack of bandwidth. It is MIT licensed. Take it, fork it, build it, sell it. No need to ask.

## The idea in one line

A robot arm on a wheeled base where a generative AI model (LLM/VLM) is the decision maker, choosing and sequencing a library of pre-coded skills (drive to X, find Y, grasp Z, place, open) to carry out plain-language instructions.

## Why it might matter

Classical robots need every task scripted. An LLM can turn "clean up the table" into a plan over skills the robot already knows how to do reliably. Cheap mobile manipulators (low-cost arms on hobby or warehouse bases) plus open VLMs make this buildable by a small team now.

## What exists

Spec only (see SPEC.md). No code. Note the prior art below: the core concept is already patented and widely published, so any new value is in a specific improvement, not the general idea.

## What's left

- Pick a niche where it earns money (hospital supply runs, retail restocking, lab sample handling, home assistance).
- Build the skill library and the LLM planner on top of an existing stack (ROS 2, MoveIt, Nav2).
- Find a technical improvement that is genuinely new (see "Possible openings" in SPEC.md).
- Freedom-to-operate review by a patent attorney before selling anything.

## Where to start

Read the SayCan paper and Google's patent US20230311335A1 first. Then reproduce TidyBot-style behaviour on a cheap arm + base in simulation before buying hardware.

## Notes/pitfalls

- Google's "Natural language control of a robot" patent family covers LLM + skill library + affordance scoring on a robot. Design around its claims or license it if you commercialize.
- LLMs propose infeasible or unsafe actions. You need a hard safety layer between the planner and the motors.
- Grasp failure recovery, not planning, is usually what breaks in the real world.
- This handover is not legal advice. The patent search behind it was a quick web search, not a professional search.
