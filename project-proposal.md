Degree level: PhD

# Project proposal

## 1. The problem and goal

(Explain the problem you want to solve and why it matters. Describe what you
want to improve, how you will measure improvement, and which limits or
requirements must be respected. A short example can help someone outside your
field understand the problem.)

**The problem and goal in my project:**

Designing a tabletop optical setup (a beam expander, an interferometer, a
frequency-conversion stage) is still a manual, expert-driven loop: choose a
layout, pick real catalog parts, place them, check the beam, check how
sensitive the result is to small misalignments, and repeat. A single mistake
(a beam clipping a lens edge, an interferometer arm that does not close, a
layout that drifts out of spec when a mount is nudged by a few micrometres)
often shows up only after the parts are bought and aligned. For example,
"expand a 1 mm 1064 nm beam to 5 mm, collimated, in under 20 cm, using stock
Thorlabs lenses, and keep it within spec under ±50 µm / ±0.5 mrad mount errors"
takes an experienced student hours; a bad choice takes days to discover.

The goal is an LLM agent that turns such a target specification into a
validated, tolerance-robust and buildable design, and, as the research
question, to measure **where an LLM actually adds value** in physical design
compared to classical search and optimization.

I will measure improvement on a benchmark of ~15 design tasks of increasing
difficulty (telescopes, fiber coupling, Mach–Zehnder and Michelson
interferometers, second-harmonic generation stages):

- **Success rate:** the design meets every target metric with no
  validation errors and no unresolved simulator warnings.
- **Robustness:** Monte Carlo yield (fraction of perturbed builds still in
  spec) at the stated mechanical tolerances.
- **Cost:** number of simulator calls, LLM tokens and wall-clock time per
  solved task.
- **Baselines:** random/grid search and a numeric optimizer (CMA-ES or
  Bayesian optimization) over a fixed topology, plus a hybrid where the agent
  picks the topology and parts and the optimizer tunes distances.

Limits that must be respected: only real, purchasable catalog parts; physical
validity (no clipping, closed interferometer loops, consistent angles of
incidence); a footprint that fits a 3D-printer bed; and honesty about the
simulator: nonlinear conversion is not yet simulated end to end, so tasks that
depend on it are scored only on phase matching until that layer is complete.

## 2. The simulator

(Describe a program or model that could simulate your system and calculate what
happens under different conditions. Explain what inputs it takes and what
results it produces. State whether it can run on your computer and accept inputs
in a text file, such as JSON or YAML. If you have not chosen a simulator yet,
name an option you want to explore and explain what you need to check.)

**The simulator in my project:**

AutoOptics, a Python package I have been developing. A setup is written as a
directed graph in a JSON file (optical elements as nodes, beam segments as
edges, with catalog part numbers, distances and deflection angles). The
simulator then runs:

- **Layer 1:** ABCD matrices, Gaussian-beam propagation and Jones polarization
  (beam size, waist position, divergence, polarization at every element).
- **Layer 2:** scalar wave optics (angular spectrum) where coherence matters,
  for example fringe visibility and phase in interferometers.
- **Layer 3 (partial):** crystal optics and phase matching (Δk, coherence
  length) for χ⁽²⁾ crystals through Lightwave Explorer; full nonlinear
  conversion is the open frontier.
- **Tolerance engine:** sensitivity Jacobians and covariance propagation of
  mechanical errors, cross-checked by Monte Carlo, giving a variance budget
  and yield.
- **Validation:** geometric checks (loop closure, angle-of-incidence
  consistency, beam obstruction) with error/warning severities.

Outputs are JSON metrics, a markdown report with layout figures, and
optionally a 3D-printable STEP baseplate with built-in flexure stages and an
alignment plan. It runs on my laptop (Python 3.11, no GPU) against a local
database of 34k Thorlabs products. Most of the simulator already exists, so
the course project concentrates on the agent and its evaluation.

## 3. The agent's role

(Explain what the language-model-based agent will decide or change, and how it
will use the simulator's results to choose its next action. If known, list the
variables it may change and their allowed ranges. Explain what you expect the
language model to contribute: could a simple program that tries different
options and picks the best result achieve the same goal? Include one question
you still need to answer.)

**The agent in my project:**

The agent edits the JSON design graph through tools (look up a catalog part,
add/remove/replace an element, set a distance or angle, validate, simulate,
run tolerance analysis). It decides:

- **Topology:** which elements and in what order (e.g. Galilean vs Keplerian
  telescope, one vs two fold mirrors, where to put a waveplate).
- **Parts:** which catalog items to use (focal length, diameter, coating,
  crystal), from ~34k products.
- **Continuous parameters:** element spacings (0–500 mm), fold angles
  (0–90°), crystal angles, and which elements get adjustable flexure stages.

After each change it reads the simulator's metrics, validation errors and
warnings, and the tolerance report (which mount errors dominate the variance
budget), then decides the next edit, stopping only when every metric is in
spec and no errors or warnings remain.

What I expect the language model to contribute: the choices a brute-force
search handles badly. Topology and part selection are discrete and
combinatorial, and they benefit from domain heuristics (stored in a versioned
`skills.md` file the agent reads) and from understanding *why* a design
failed ("the beam clips L2: use a 1-inch lens or move it closer"). Pure
numeric tuning of distances is probably done better and cheaper by an
optimizer, so the agent will be allowed to call one. The benchmark compares
agent-only, optimizer-only and the hybrid to test this directly.

Open question: does the agent's ability to update its own heuristics file
across tasks measurably improve success and cost on unseen tasks, or does it
only restate textbook rules it already knew?

## Explain your project to someone outside your field

(Now describe your whole project again, as if you were explaining it to someone
from another field who has no background in your topic. For example, if you
study chemical engineering, imagine explaining it to a physicist; if you study
aeronautical engineering, imagine explaining it to someone in medicine; if you
study data science, imagine explaining it to a materials engineer. In one or two
short paragraphs, connect all three parts: the problem you want to solve, how
the simulator helps you test ideas, and what the agent will decide or change.
Use everyday language, explain any technical terms you need, and avoid
unexplained abbreviations. Do not simply copy your answers above.)

**My project in simple words:**

In a laser lab, experiments are built from small parts (lenses, mirrors,
special crystals) bolted to a table so that a laser beam travels through them
in the right order and comes out with the right size and direction. Choosing
and placing these parts is a bit like building a model from thousands of
different Lego bricks with only a picture of the result: an expert tries a
layout, checks it, and fixes it, often for hours, and some mistakes only
appear once the real parts are on the table, or when a part shifts by the
width of a hair and the whole experiment stops working.

I have written a program that predicts, on a computer, what the beam will do
for any proposed layout, and how badly small shifts of the parts would spoil
it. In this project an AI assistant uses that program like a test bench: it
picks real parts from a manufacturer's catalog, arranges them, runs the
prediction, reads what went wrong, and changes the design until it meets the
requested goal and stays robust. I will compare it with simpler computer
search methods to find out which design decisions really need the assistant's
judgement and which a plain search does just as well.
