# **EM33 Cart-Pole Controller (Taichi + GA) — Minimal README**

-> warning! it is normal both on training and on inference that requires > 1m to setup the taichi kernels and initialisation. wait .

This project implements an **emergent EM33 cellular-automaton controller** for the cart-pole environment, using:

* **Taichi** for massively parallel, GPU-efficient simulation
* **General GA** (vectorized NumPy) for evolving CA rules + tapes
* **Fully batched environment** (A agents evaluated per episode)
* **Episodic fitness** = number of alive steps
* **Hard episode termination** (dead agents never revive)

Everything is deterministic-free (no fixed seeds).

---

## **Overview**

### **1. Genome**

Each agent has two learnable structures:

* **Rule table** `R`: shape `(27,)`, values in `{0,1,2}`
* **Program tape** `S_init`: shape `(N,)`, values in `{0,1,2}`

  * On every step, **input cells** are overwritten by encoded environment observations
  * Non-input cells evolve according to the CA rule

### **2. Cellular Automaton (EM33)**

* 3-state 1-D CA (states 0,1,2)
* Neighborhood encoded as `left*9 + center*3 + right` → rule index `0..26`
* T steps per control-step (`T=24` by default)

### **3. Environment**

A vectorized cart-pole physics implemented in Taichi:

* Euler integration
* Done for all A agents simultaneously
* Agent dies if |x| > threshold or |θ| > threshold
* Dead agent stays dead until next episode

### **4. GA Workflow**

1. **ask()** → get current population (R, S_init)
2. **run_episode()** → Taichi sim, returns fitness
3. **tell(fitness)** → GA evolves next generation

Repeat for many generations.

---

## **Running**

### **Training**

Run the notebook cell that:

* Instantiates GA
* Instantiates `EM33CartPoleSystem`
* Loops: GA ask → Taichi simulation → GA tell
* Prints average & best fitness per generation

### **Inference + Visualization**

A separate cell provides a **pygame live visualization**:

* Loads the best agent from the last train step
* Runs a single-agent Taichi simulation in real time
* Renders cart, pole, and status
* Controls:

  * **ESC**: quit
  * **R**: reset episode
  * **SPACE**: pause

---

## **Files / Structure**

* **training cell** — GA + Taichi environment
* **inference cell** — pygame live viewer
* **general_ga.py** — standalone GA engine
* **EM33CartPoleSystem** — Taichi CA + physics simulator

---

## **Fitness**

Fitness per agent = number of steps alive in the episode.
Dead ⇒ fitness stops increasing.
Alive ⇒ increments every step.

---

## **Notes**

* All random processes use Taichi/NumPy RNG without seeds.
* Everything is GPU-parallel and safe for large populations.
* Model and environment are strictly separated inside Taichi.
