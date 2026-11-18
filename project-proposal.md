# Project proposal

## 1. The problem and goal

**The problem and goal in my project:**

I want to design a planar steel truss, such as a 6–10 panel bridge or roof truss, that carries given loads at the **lowest possible mass**. The **requirements** are: the stress in every member stays below the allowable stress (yield ÷ safety factor 1.5), slender bars in compression don't buckle (Euler check), and midspan deflection stays at or below L/360. Bar cross-sections must come from a discrete catalogue of standard tube sizes. **Improvement is measured** as mass saved compared with a baseline design where every member has the same size, provided all constraints still pass.

Example: a footbridge truss where every bar is the same thick tube is safe but wasteful. Most bars barely carry any load, so the goal is to find out which bars can be thinner without the bridge failing.

## 2. The simulator

**The simulator in my project:**

I'll use a linear-elastic 2D frame/truss finite-element solver. The candidates are **anaStruct** (open source, pip-installable) or a ~150-line direct-stiffness solver I write myself and verify against textbook problems.

- **Inputs:** node coordinates, member connections, cross-section area and moment of inertia per member, material (E, yield stress), supports and loads.
- **Outputs:** axial force and stress per member, node displacements, total mass, and the utilisation ratio (stress ÷ allowable) per member.

It runs locally in milliseconds, is fully deterministic, and reads the design from a **JSON file**. It's easy to unit test against hand-calculated trusses, for example a simple 3-bar truss solved by the method of joints. *Still to check:* whether anaStruct's buckling and section handling is enough, or whether I add the Euler check myself.

<div style="page-break-before: always"></div><br>

## 3. The agent's role

**The agent in my project:**

The agent proposes a design (a JSON with a section size for each member, and optionally the height of the truss), runs the solver through an MCP tool, reads which members are overstressed, underused or close to buckling and whether deflection is binding, and then adjusts.

- **Variables:** section index per member (from about 8 catalogue sizes); truss height 0.1–0.25 × span; optionally removing near-zero-force members.
- **What the LLM adds:** a classic optimiser (fully stressed design, a genetic algorithm) could size the members, and I'll use it as the **comparison baseline in my eval**. The LLM's value is in the decisions that are hard to encode: interpreting *why* a constraint is binding (for example, deflection is governing, so making the truss taller helps more than making the bars thicker), grouping symmetric members, deciding when to change the geometry rather than the sizes, and explaining the final design in engineering terms. With about 30 members and 8 sizes, brute force is impossible (8^30 combinations).
- **Open question:** does the agent reach a feasible design within ~20 solver calls more reliably than a naive random search, and how close does it get to the classical optimiser's mass?

## Explain your project to someone outside your field

**My project in simple words:**

Picture a bridge built from straight metal bars joined into triangles. If every bar is made thick, the bridge is strong but heavy and expensive. Most bars carry very little load, though, so some can be thinner. The trick is knowing which ones, without letting any bar bend, break, or let the bridge sag too much. A simulator is a computer program that calculates, in a fraction of a second, how hard every bar is being pulled or pushed and how much the bridge sags.

The AI assistant acts like a junior engineer using that program. It suggests bar thicknesses, runs the calculation, looks at which bars are overworked or barely used, and makes changes, for example "this bar is close to breaking, make it thicker" or "the whole bridge sags, make it taller instead". It repeats this until it finds the lightest bridge that still passes every safety check. The calculations are always done by the trusted program, never guessed by the AI.
