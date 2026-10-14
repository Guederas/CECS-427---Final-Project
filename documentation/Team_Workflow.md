# CECS 427 Final Project — Team Work Division & Workflow

## Project: Botnet Propagation & Network Resilience Simulator

### Project Goal

Build an interactive application that models how malware spreads through a computer network, gradually forming a botnet. The simulator will compare different network structures, show how compromised devices spread malware over time, simulate service overload from the growing botnet, and test different defense/recovery strategies.

The application will use:

- **Python + NetworkX** for graph generation, algorithms, simulation, and analysis
- **Flask or FastAPI** for the backend API
- **JavaScript + Cytoscape.js** for the interactive network visualization
- **Chart.js** for charts and simulation statistics
- **GitHub** for source control and team collaboration
- **pytest** for backend testing

The goal is to have the project **feature-complete by November 22**, leaving the final week of November for testing, documentation, results, and presentation preparation.

---

# 1. Team Work Division

## Person 1 — Network Generation & Topology

### What are you going to be doing?

You will build the network structures that every other part of the project uses.

Your job is to create the different graph models and make sure they can be generated consistently from user-selected parameters.

You will probably implement:

- Erdős–Rényi random networks
- Watts–Strogatz small-world networks
- Barabási–Albert scale-free networks
- Network size controls
- Average degree / connection parameters
- Random seeds for reproducibility
- Basic network statistics

You should provide one clean interface that the rest of the backend can call, such as:

```python
generate_network(...)
```

The function should return a **NetworkX graph** that the simulation, analysis, defense, and visualization systems can all use.

### What tools will you probably use?

- Python
- NetworkX
- NumPy / Python `random`
- pytest
- Git / GitHub
- VS Code or another Python IDE

### Main files

```text
backend/network_generator.py
tests/test_network_generator.py
```

### Important outputs

Your module should be able to provide:

- Graph object
- Number of nodes
- Number of edges
- Average degree
- Graph density
- Connected components
- Network type and parameters used

### Who will you work closely with?

**Person 6**, because the generated graph must be converted into data that Cytoscape.js can display.

You will also work with **Person 5** to expose graph generation through the backend API.

---

# 2. Person 2 — Malware Propagation Simulation

## What are you going to be doing?

You will build the core malware-spread simulation.

Each node will represent a device. Devices can change state over time.

Initial states will probably include:

- `vulnerable`
- `compromised`
- `patched`
- possibly `quarantined`

The simulation will begin with one or more compromised devices ("Patient Zero"), then malware will spread through connected nodes according to a user-selected compromise probability.

Compromised devices may also recover/be patched based on another probability.

You will probably build the main simulation function:

```python
run_simulation(...)
```

The simulation should update devices in **discrete time steps**.

Important: all state changes for a time step should be calculated first and then applied simultaneously. Newly compromised nodes should not immediately infect another device during the same step.

### What tools will you probably use?

- Python
- NetworkX
- NumPy / Python `random`
- pytest
- Git / GitHub

### Main files

```text
backend/propagation.py
tests/test_propagation.py
```

### Important test cases

Your module should verify cases such as:

- Infection probability = 0 → malware never spreads
- Infection probability = 1 and recovery = 0 → all reachable vulnerable nodes eventually become compromised
- Recovery probability = 0 → compromised nodes never recover
- Malware cannot cross disconnected network components
- Fixed random seeds reproduce the same simulation

### Who will you work closely with?

**Person 4**, because defenses will directly affect the propagation model.

You will also work with **Person 5** because the API needs to return the simulation history to the frontend.

---

# 3. Person 3 — Network Analysis & Metrics

## What are you going to be doing?

You will analyze the graph and calculate the network-science metrics used in our experiments.

This is one of the most important roles for connecting the project to **CECS 427 concepts**.

You will probably implement:

- Degree centrality
- Betweenness centrality
- Shortest paths
- Connected components
- Largest connected component
- Average path measurements
- Distance from a vulnerable device to the nearest compromised bot
- Rankings of influential nodes

A useful common interface would be:

```python
calculate_metrics(...)
```

You should also implement the original **bot proximity** idea as a metric:

> For each clean device, how many hops away is the nearest compromised bot?

That lets us measure how the average "distance to a bot" changes as the botnet spreads.

### What tools will you probably use?

- Python
- NetworkX
- Pandas
- NumPy
- pytest
- Git / GitHub
- Jupyter Notebook may be useful for exploring results

### Main files

```text
backend/analysis.py
tests/test_analysis.py
```

### Important outputs

Examples:

- Highest-degree node
- Highest-betweenness node
- Average distance to nearest bot
- Largest connected component size
- Number of connected components
- Network density
- Average degree
- Percentage of nodes compromised

### Who will you work closely with?

**Person 5**, because your metrics will be used heavily in the experiment framework and final results.

You will also work with **Person 4** because centrality will determine which nodes are targeted by some defenses.

---

# 4. Person 4 — Defense, Quarantine & Recovery Strategies

## What are you going to be doing?

You will implement the different defensive strategies we want to compare.

The goal is not only to simulate botnet growth, but also to answer:

> Which network defense strategy is most effective?

Initial strategies should include:

### Random patching

Randomly select a percentage of nodes and protect them before the simulation.

### Targeted patching

Protect the most connected or most central nodes first.

Possible rankings:

- Degree centrality
- Betweenness centrality

### Quarantine

When a compromised node is detected, isolate it from the network by removing or disabling its connections.

### Targeted cleanup

Prioritize cleaning compromised nodes that are the most structurally important.

Your module should expose something like:

```python
apply_defense(...)
```

### What tools will you probably use?

- Python
- NetworkX
- NumPy / Python `random`
- pytest
- Git / GitHub

### Main files

```text
backend/defenses.py
tests/test_defenses.py
```

### Important experiments

Compare the same network and malware settings using:

- No defense
- Random patching
- Targeted patching
- Quarantine
- Targeted cleanup

The important rule is that different defenses should use the **same defense budget** when possible.

Example:

> Random patching protects 10% of nodes, so targeted patching should also protect 10%.

That makes comparisons fair.

### Who will you work closely with?

**Person 2**, because propagation and defense logic directly affect each other.

You will also work with **Person 3** for centrality-based defenses.

---

# 5. Person 5 — Backend API, Service Resilience & Experiments

## What are you going to be doing?

You will connect the backend modules together and expose them to the frontend.

You will probably create the backend web server with **Flask or FastAPI**.

Possible endpoints:

```text
GET  /api/health
POST /api/network
POST /api/simulation
POST /api/experiment
```

You will also implement the abstract service-overload model.

Example:

```python
bot_load = compromised_devices * load_per_bot

total_load = legitimate_load + bot_load
```

The simulated service might have states such as:

- Healthy
- Degraded
- Down

This is only a mathematical simulation. We are **not generating real DDoS traffic**.

You will also build the repeated experiment system.

For example:

```python
run_experiment(
    network_type,
    defense_strategy,
    trials=50
)
```

This should run many simulations and aggregate results.

### What tools will you probably use?

- Python
- Flask **or** FastAPI
- NetworkX
- Pandas
- NumPy
- pytest
- Postman or Bruno for API testing
- Git / GitHub

### Main files

```text
backend/app.py
backend/resilience.py
backend/experiments.py

tests/test_resilience.py
```

### Important experiment outputs

Across repeated simulations, record things like:

- Peak number of compromised devices
- Percentage ever compromised
- Time to peak
- Time to containment
- Total simulation duration
- Service downtime
- Average bot proximity
- Largest connected component
- Defense success rate

For repeated trials, calculate:

- Mean
- Median
- Standard deviation

### Who will you work closely with?

**Person 3** for metrics and experiment analysis.

You must also work very closely with **Person 6**, because your API is the bridge between Python and the frontend.

---

# 6. Person 6 — Frontend, Cytoscape.js & Visualization

## What are you going to be doing?

You will build the interactive application the professor and class actually see.

The main network visualization will use **Cytoscape.js**.

The frontend should let users:

- Select the network type
- Choose number of devices
- Adjust malware spread probability
- Adjust recovery probability
- Generate a network
- Click a node to select Patient Zero
- Select a defense strategy
- Start the simulation
- Pause the simulation
- Step through the simulation
- Reset the simulation
- Change animation speed

The Cytoscape graph should visually distinguish states such as:

- Vulnerable
- Compromised
- Patched
- Quarantined

The graph should keep the same node positions during the simulation so users can visually follow the malware spread.

You will also use **Chart.js** for normal charts such as:

- Vulnerable / compromised / patched devices over time
- Service load over time
- Comparison of defense strategies
- Experiment results

### What tools will you probably use?

- JavaScript
- HTML
- CSS
- Cytoscape.js
- Chart.js
- Vite
- Browser developer tools
- Fetch API
- Git / GitHub

### Main files

```text
frontend/index.html

frontend/src/main.js
frontend/src/graph.js
frontend/src/simulation.js
frontend/src/controls.js
frontend/src/charts.js
frontend/src/styles.css
```

### Important frontend responsibility

Do **not wait for the backend** before starting.

Build the Cytoscape visualization using fake/mock JSON first.

Once the backend API works, replace the fake data with real API calls.

### Who will you work closely with?

**Person 1** for NetworkX → Cytoscape graph data.

**Person 5** for backend API integration.

---

# Shared Architecture

The project should roughly follow this structure:

```text
                    USER
                      │
                      ▼
           ┌─────────────────────┐
           │      Frontend       │
           │                     │
           │ Cytoscape.js        │
           │ Chart.js            │
           │ HTML/CSS/JavaScript │
           └──────────┬──────────┘
                      │
                 JSON / HTTP
                      │
                      ▼
           ┌─────────────────────┐
           │    Flask/FastAPI    │
           │       Backend       │
           └──────────┬──────────┘
                      │
       ┌──────────────┼────────────────┐
       │              │                │
       ▼              ▼                ▼
 Network Models   Propagation      Defenses
       │              │                │
       └──────────────┼────────────────┘
                      │
                      ▼
              Network Analysis
                      │
                      ▼
            Resilience / Experiments
```

---

# Recommended Repository Structure

```text
botnet-simulator/
│
├── backend/
│   ├── app.py
│   ├── network_generator.py
│   ├── propagation.py
│   ├── analysis.py
│   ├── defenses.py
│   ├── resilience.py
│   └── experiments.py
│
├── frontend/
│   ├── index.html
│   ├── src/
│   │   ├── main.js
│   │   ├── graph.js
│   │   ├── simulation.js
│   │   ├── controls.js
│   │   ├── charts.js
│   │   └── styles.css
│   └── package.json
│
├── tests/
│   ├── test_network_generator.py
│   ├── test_propagation.py
│   ├── test_analysis.py
│   ├── test_defenses.py
│   └── test_resilience.py
│
├── results/
├── requirements.txt
├── README.md
└── .gitignore
```

---

# GitHub Workflow

## Main Branch

`main` should always contain a reasonably stable version of the project.

Avoid making large changes directly on `main`.

Each person should use a feature branch.

Examples:

```text
feature/network-models
feature/propagation
feature/analysis
feature/defenses
feature/api-resilience
feature/frontend
```

Workflow:

```text
Create branch
    ↓
Implement feature
    ↓
Write/update tests
    ↓
Push branch
    ↓
Open Pull Request
    ↓
At least one teammate reviews it
    ↓
Merge into main
```

Do not wait until the end of the semester to merge all six modules.

---

# Integration Partners

Even though everyone owns a module, we should work in pairs for integration.

## Pair 1

**Person 1 ↔ Person 6**

```text
NetworkX graph
      ↓
JSON
      ↓
Cytoscape visualization
```

## Pair 2

**Person 2 ↔ Person 4**

```text
Malware propagation
      +
Defense / recovery
```

## Pair 3

**Person 3 ↔ Person 5**

```text
Network metrics
      +
Experiment framework / API
```

These are not separate teams. Everyone still collaborates, but these pairings identify the closest dependencies.

---

# Development Timeline

## Week 1 — October 10–18

### Goal: Architecture + basic independent modules

Before doing heavy development, agree on the shared interfaces.

At minimum define:

```python
generate_network(...)
run_simulation(...)
calculate_metrics(...)
apply_defense(...)
calculate_service_load(...)
```

Also agree on the JSON structure between Python and JavaScript.

Example:

```json
{
  "step": 4,
  "nodes": [
    {"id": "0", "state": "vulnerable"},
    {"id": "1", "state": "compromised"},
    {"id": "2", "state": "patched"}
  ],
  "metrics": {
    "bots": 18,
    "serviceLoad": 76
  }
}
```

### By October 18

We should have:

- GitHub repository created
- Folder structure finalized
- README skeleton
- Python environment working
- Frontend/Vite project working
- Shared interfaces agreed upon
- Each person actively implementing their core module
- Person 6 displaying a fake/mock graph in Cytoscape

---

# Week 2 — October 19–25

## Goal: First integrated working simulation

Connect:

```text
Frontend
   ↓
Generate Network
   ↓
Python / NetworkX
   ↓
JSON
   ↓
Cytoscape.js
```

Also connect:

```text
Frontend
   ↓
Run Simulation
   ↓
Python propagation engine
   ↓
Simulation history
   ↓
Cytoscape animation
```

### By October 25

The application should be able to:

1. Generate a network
2. Display it in Cytoscape
3. Select Patient Zero
4. Run malware propagation
5. Animate changing device states
6. Reset and run again

It does **not need to look pretty yet**.

A working ugly prototype is more valuable than a beautiful incomplete application.

---

# Week 3 — October 26–November 1

## Goal: Metrics + service resilience

Add:

- Network statistics
- Degree centrality
- Betweenness centrality
- Connected components
- Bot proximity
- Peak infections
- Service load
- Healthy / degraded / down status

Frontend should begin showing charts and a statistics panel.

### By November 1

The entire basic simulation pipeline should work.

---

# Week 4 — November 2–8

## Goal: Defense strategies

Implement and integrate:

- No defense
- Random patching
- Targeted degree-based patching
- Quarantine
- Targeted cleanup if time allows

### By November 8

We should be able to run the **same network and malware parameters** with different defenses and clearly see different outcomes.

---

# Week 5 — November 9–15

## Goal: Experiments and results

Build the repeated experiment system.

Compare:

### Network Types

- Random
- Small-world
- Scale-free

### Defense Strategies

- None
- Random patching
- Targeted patching
- Quarantine

Run repeated simulations with controlled parameters and seeds.

For example:

```text
50 trials per configuration
```

Collect:

- Peak bots
- Total percentage compromised
- Time to peak
- Outbreak duration
- Service downtime
- Bot proximity
- Largest connected component

### By November 15

We should have actual preliminary results that can be used in our report and presentation.

---

# Week 6 — November 16–22

# FEATURE FREEZE

This is the final implementation week.

Focus on:

- Fixing bugs
- Integration
- Unit tests
- Input validation
- Performance
- UI cleanup
- Reproducibility
- Documentation

Test different sizes and edge cases:

```text
10 nodes
50 nodes
100 nodes
500 nodes
```

Test:

```text
infection = 0
infection = 1
recovery = 0
disconnected networks
hub as Patient Zero
low-degree node as Patient Zero
different defense strategies
```

### November 22 = Feature Complete

After November 22:

> Do not add major new features unless something required is missing.

From this point onward, prioritize stability.

---

# Week 7 — November 23–29

## Goal: Testing, documentation, report, final results

Tasks:

- Run all automated tests
- Clone the repository onto another computer
- Verify installation instructions
- Verify frontend/backend setup
- Finalize README
- Finalize dependencies
- Save experiment data
- Create final comparison charts
- Write project report
- Document each person's contributions
- Cite external libraries/resources

### By November 29

The project should be essentially finished.

---

# November 30 – December 2

## Goal: Presentation and demo rehearsal

Prepare a reliable demo configuration.

Example:

```text
Network:
Scale-free

Nodes:
100

Initial compromised nodes:
1

Compromise probability:
0.25

Recovery probability:
0.05

Defense:
Targeted patching
```

Use a **fixed random seed** so the live demonstration behaves predictably.

Practice:

1. Explain problem
2. Explain network representation
3. Explain propagation model
4. Explain network concepts
5. Demonstrate simulator
6. Show defenses
7. Show experimental results
8. Discuss limitations

Everyone should understand the overall application, not only their own module.

---

# December 3–4

## Emergency / Final Buffer

Only use this time for:

- Bugs
- Merge problems
- README issues
- Missing citations
- Presentation timing
- Small UI fixes
- Submission problems

Avoid major new features.

---

# Major Milestones

| Deadline | Milestone |
|---|---|
| **October 18** | Architecture + independent modules underway |
| **October 25** | First integrated simulation |
| **November 1** | Metrics + service resilience |
| **November 8** | Defense strategies |
| **November 15** | Experiment framework + preliminary results |
| **November 22** | **FEATURE FREEZE** |
| **November 29** | Code, testing, README, report complete |
| **December 1–2** | Presentation rehearsals |
| **December 3–4** | Final buffer |

---

# Suggested Weekly Team Workflow

## Beginning of the Week

Have a short meeting and answer:

- What did I finish?
- What am I working on this week?
- What is blocking me?
- Does someone else depend on my code?
- Do I depend on someone else's code?

Keep this meeting short.

---

## Midweek Integration Check

Everyone should pull the newest `main` branch.

Verify:

- Backend still runs
- Frontend still runs
- Tests still pass
- Your module still integrates correctly

Do not wait until Sunday to discover integration problems.

---

## End of Week

- Finish scheduled milestone
- Push feature branches
- Open pull requests
- Review teammates' code
- Merge stable work into `main`
- Run tests
- Verify application still runs

---

# Recommended Git Tags / Versions

We can optionally tag stable milestones:

```text
v0.1-network
v0.2-propagation
v0.3-integrated
v0.4-defense
v0.5-experiments
v1.0-final
```

This gives us stable restore points if a later change causes problems.

---

# Important Team Rules

## 1. Agree on interfaces early

Do not independently change function arguments or JSON structures without telling the people who depend on them.

---

## 2. Integrate continuously

Do not build six independent projects and attempt to connect them in late November.

---

## 3. Tests belong to the person writing the module

There should not be a separate "testing person."

If you build a feature, you should also build its tests.

---

## 4. Documentation belongs to everyone

Each person should document:

- Functions
- Inputs
- Outputs
- Assumptions
- Tests
- External resources/libraries used

---

## 5. Keep the DDoS simulation abstract

We are modeling service overload mathematically.

We do **not** need to send real malicious traffic or implement actual attack software.

---

## 6. Keep experiments fair

When comparing networks or defenses:

- Use comparable network sizes
- Use similar average degrees when possible
- Use the same malware parameters
- Use equal defense budgets
- Use controlled random seeds
- Run multiple trials

---

## 7. MVP first, extra features second

Our minimum viable project is:

1. Generate graph
2. Visualize graph
3. Select Patient Zero
4. Simulate malware spread
5. Animate propagation
6. Calculate network metrics
7. Simulate service overload
8. Apply at least one defense
9. Compare outcomes
10. Reset and run another scenario

Everything beyond that should be added **after this version works reliably**.

---

# Final Research Direction

A strong main question for the project is:

> **How does network topology influence botnet propagation, and how effectively can targeted defensive strategies reduce malware spread and preserve service availability?**

Supporting questions can include:

- Which network structure allows malware to spread fastest?
- Does compromising a central node lead to a larger botnet?
- How quickly does average distance to the nearest bot decrease?
- Is targeted patching more effective than random patching?
- How does quarantine affect network connectivity?
- Which network structure is most resilient after compromised devices are isolated?
- At what point does the growing botnet cause simulated service failure?

These questions should guide our implementation and final experiments.
