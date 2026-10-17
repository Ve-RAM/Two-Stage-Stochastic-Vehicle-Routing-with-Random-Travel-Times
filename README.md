# Two-Stage Stochastic Vehicle Routing with Random Travel Times

## Overview

This project develops a **two-stage stochastic optimization model for vehicle routing under uncertain travel times**. The objective is to select a route that minimizes total transportation and operational costs when traffic conditions are unknown at the time of planning.

The problem is formulated as a two-stage stochastic mixed-integer linear program (MILP). First-stage decisions determine a single directed route visiting all service locations. Once travel times are realized, second-stage decisions determine how much work to perform directly, how much to outsource, and whether to use additional working time.

The implementation combines historical data processing, scenario-based stochastic programming, mathematical optimization, decomposition algorithms, and independent solution validation in Python.

## Key Features

- **Historical data processing:** Extracts and filters NYC Yellow Taxi trip records to estimate travel-time distributions for directed arcs between service locations.
- **Scenario generation:** Constructs synthetic travel-time scenarios through independent sampling from historical arc-specific distributions.
- **Two-stage stochastic programming:** Separates route selection from operational decisions made after uncertainty is revealed.
- **MILP formulation:** Models route selection using binary variables and the Dantzig–Fulkerson–Johnson (DFJ) subtour-elimination constraints.
- **Integer L-shaped decomposition:** Implements a branch-and-cut-style solution procedure using scenario-level recourse problems, dual-based Benders cuts, and integer optimality cuts.
- **Alternative formulations:** Compares the extensive-form model with Integer L-shaped decomposition and independently checks the routing formulation against Miller–Tucker–Zemlin (MTZ) constraints.
- **Independent verification:** Audits model constraints, route feasibility, integrality, objective values, recourse decisions, and optimization bounds.
- **Out-of-sample evaluation:** Compares routing decisions using historical data from January and February 2015.
- **Reproducible experiments:** Uses fixed random seeds, data fingerprints, cached preprocessing results, and saved experiment outputs.

## Mathematical Formulation

### First stage: Route selection

Let \(V=\{0,1,\ldots,10\}\) denote the depot and ten service locations, and let \(A\) be the set of directed arcs between distinct nodes.

The binary decision variable \(x_{ij}\) indicates whether the route uses arc \((i,j)\):

\[
x_{ij}\in\{0,1\}.
\]

The routing constraints enforce exactly one incoming and one outgoing arc at each node. DFJ subtour-elimination constraints ensure that the resulting solution is a single directed tour visiting every location and returning to the depot.

The selected route is shared across all travel-time scenarios, reflecting the requirement to commit to a route before observing traffic conditions.

### Second stage: Operational recourse

For each scenario \(s\), the model decides how to allocate service demand between direct work and outsourcing, and how much additional working time to use.

The recourse decisions include:

- \(u_i^s\): direct work performed at location \(i\).
- \(r_i^s\): outsourced work at location \(i\).
- \(o^s\): ordinary overtime.
- \(e^s\): emergency additional time.

Demand must be fully covered, and the sum of travel time and direct work must respect the available working horizon, accounting for permitted additional time.

The recourse objective minimizes outsourcing and additional-time costs. The resulting expected operational cost is combined with the first-stage routing cost.

### Stochastic objective

For a finite sample of \(K\) equally likely scenarios, the extensive-form objective is

\[
\min_{x,y_1,\ldots,y_K}
\left\{
c^\top x+
\frac{1}{K}\sum_{s=1}^{K}q^\top y_s
\right\},
\]

subject to the routing constraints and the scenario-specific recourse constraints.

This formulation captures the trade-off between selecting a route with favorable transportation costs and choosing a route that performs well under uncertain travel conditions.

## Solution Methodology

The notebook implements and compares the following approaches:

1. **Extensive-form MILP:** Solves the deterministic equivalent containing the routing decisions and all scenario-specific recourse variables.
2. **Integer L-shaped decomposition:** Iteratively solves a master problem and evaluates candidate routes through scenario-level linear programming subproblems.
3. **Dual-based Benders cuts:** Uses recourse dual information to construct cuts that strengthen the master problem.
4. **Integer optimality cuts:** Adds valid cuts for evaluated binary routing solutions.
5. **MTZ formulation benchmark:** Independently checks the DFJ routing formulation on the 50-scenario instance.

The implementation also includes an independent piecewise evaluation of the recourse problem, used to check operational costs and decisions against the optimization model.

## Data and Experimental Design

The experiments use NYC Yellow Taxi trip records from January and February 2015.

The network consists of ten service locations in Manhattan and a depot at the Javits Center. Historical trips are filtered by pickup date, weekday, pickup time, trip duration, and trip distance. Directed arc-specific travel-time samples are then constructed from the filtered records.

The main experiment uses synthetic scenarios sampled with replacement from the historical travel-time pools.

| Experiment | Scenarios | Purpose |
|---|---:|---|
| Extensive-form MILP | 50 | Baseline optimization |
| Integer L-shaped | 50 | Decomposition consistency |
| Extensive-form MILP with MTZ | 50 | Independent routing-formulation check |
| Integer L-shaped | 200 | Main stochastic optimization |
| Expected-value model | 1 average profile | Comparison with deterministic planning |
| Out-of-sample evaluation | 1,000 shared scenarios | Temporal validation using February data |

All experiments use fixed random seeds for reproducibility.

## Results

### Main optimization results

The following values are from the notebook's recorded execution.

| Method | Scenarios | Objective | Optimality gap |
|---|---:|---:|---:|
| Extensive-form MILP | 50 | 56.199410 | 0% |
| Integer L-shaped | 50 | 56.199410 | 0% |
| Extensive-form MILP with MTZ | 50 | 56.199410 | 0% |
| Integer L-shaped | 200 | 49.030852 | 0% |

The extensive-form and Integer L-shaped formulations produce matching objectives on the 50-scenario instance. The MTZ benchmark also produces the same objective, providing an additional check of the routing formulation.

For the 200-scenario instance, the Integer L-shaped algorithm closes the reported optimality gap and returns the following route:

**Depot → Madison Square Garden → Times Square → Grand Central Terminal → Rockefeller Center → New York Public Library → Union Square → Washington Square Park → South Street Seaport → New York Stock Exchange → One World Trade Center → Depot.**

### Expected-value comparison

The expected-value model optimizes a deterministic profile based on average travel times. Its objective is 47.351218, compared with 49.030852 for the stochastic model.

When the expected-value route is evaluated on the same 200 scenarios, its total cost is also 49.030852. Consequently, the reported value of the stochastic solution (VSS) is zero for this particular experiment.

This is an instance-specific result: it does not establish that deterministic planning and stochastic optimization are generally equivalent.

### Temporal validation

The notebook evaluates both routing decisions using 1,000 shared synthetic scenarios generated from February travel-time data, while keeping the January-derived routes and transportation costs fixed.

The two routes coincide in the recorded experiment, and the estimated difference in their evaluated costs is zero. Since the comparison involves identical routing decisions, this result reflects the specific data, scenario generation, and cost assumptions used in the experiment; it should not be interpreted as evidence of universal out-of-sample equivalence.

## Technologies

- **Python** — Implementation and experiment orchestration.
- **NumPy** — Numerical computations and scenario generation.
- **pandas** — Data processing and tabular analysis.
- **PyArrow** — Reading and processing Parquet data.
- **Matplotlib** — Visualizations and diagnostic plots.
- **Pyomo** — Mathematical optimization modeling.
- **HiGHS** — MILP and linear programming solver.
- **Jupyter Notebook** — Integrated documentation, modeling, computation, and reporting.

## Repository Contents

- `Practica_1_Jaime_Andrés_Jaramillo_Ramírez_Santiago_Vera_Ramírez.ipynb` — Main notebook containing data preparation, mathematical formulation, optimization algorithms, computational experiments, verification procedures, visualizations, and results.
- `README.md` — Project overview, methodology, experimental design, and reported findings.

The notebook also saves experiment outputs and diagnostic figures in a results directory during execution. The underlying Parquet datasets and generated cache files are not included as part of the notebook's documented results package.

## How to Run

### 1. Install the dependencies

```bash
pip install numpy pandas matplotlib pyarrow pyomo highspy jupyter
```

### 2. Launch Jupyter

```bash
jupyter notebook
```

### 3. Open the notebook

Open the project notebook and execute its cells in order.

The notebook downloads the required January and February 2015 NYC Yellow Taxi trip datasets from the official NYC TLC-hosted data source when needed. Ensure an internet connection is available for the initial data download.

The implementation also uses local directories for cached data and saved results. Review the path configuration in the notebook if running it in a different environment.

**Note:** Exact numerical results may depend on solver versions, execution settings, data availability, and numerical tolerances. The notebook records seeds, package versions, and data fingerprints to help verify the computational setup.

## Limitations

- Travel-time scenarios are generated by independently sampling historical observations for each directed arc; they do not reconstruct complete historical traffic days or preserve cross-arc dependence.
- The model considers a fixed network of ten service locations and a single depot.
- February validation is conditional on the historical pools, selected routes, and specified cost structure.
- The reported VSS and validation results are specific to the recorded experiments.
- Although the model incorporates stochastic travel times, it does not explicitly model time-dependent traffic, road-network paths, or correlated congestion processes.

## Authors

- Jaime Andrés Jaramillo Ramírez
- Santiago Vera Ramírez

## Academic Context

Developed as an academic project on stochastic optimization and mathematical modeling, with an emphasis on combining rigorous formulations, decomposition algorithms, computational experiments, and independent numerical verification.
