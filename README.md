# Dynamic Flood Escape

A small, reproducible grid-world assignment comparing BFS/A* with tabular
Q-Learning and PPO. Flooding is a discrete, deterministic wave through the
walkable map; it is not a fluid simulation.

## Environment

- 10×10 map, fixed walls, start `(0, 0)`, safe zone `(9, 9)`, flood source `(5, 5)`.
- State: agent row, column, and elapsed timestep. The map and flood schedule are
  fixed within each experiment.
- Actions: `0 Up`, `1 Down`, `2 Left`, `3 Right`.
- Flood scenarios: static, slow (new distance layer every 8 steps), normal (4),
  and rapid (3). A cell floods once its shortest walkable distance from the
  source is no greater than the current flood layer. The goal is protected.
- An attempted wall/boundary move consumes a timestep and costs 20 in addition
  to the step cost. Each timestep costs 1; reaching safety earns 100; being in
  a flooded cell costs 100 and terminates the episode. Episodes time out after
  100 steps.
- Shaped reward adds `0.99 * potential(next) - potential(current)`, where
  potential rewards lower Manhattan distance to safety and penalizes closeness
  to the active flood edge. The same step and terminal rewards still apply.
- BFS and A* search the same time-expanded graph, with the same known flood
  schedule. A* uses Manhattan distance to the goal (admissible because each
  move costs one and flood constraints only remove routes). Returned routes
  are replayed through the environment to score them.

## Install and run

Python 3.10+ is recommended. In PowerShell:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python dynamic_flood_escape.py --mode all
```

The default run trains Q-Learning for 2,000 episodes and PPO for 15,000
timesteps for each combination of four flood speeds and two reward types.
For a quicker smoke/demo run:

```powershell
python dynamic_flood_escape.py --mode all --q-episodes 300 --ppo-timesteps 3000 --eval-episodes 10
```

Modes:

```powershell
python dynamic_flood_escape.py --mode search
python dynamic_flood_escape.py --mode train --q-episodes 2000 --ppo-timesteps 15000
python dynamic_flood_escape.py --mode plots
python dynamic_flood_escape.py --mode demo
```

`--seed` sets the reproducible training seed (default 7). Search and flood
dynamics are deterministic. RL policies are evaluated with greedy/deterministic
actions; the random-policy baseline is averaged over the same evaluation count.

## Outputs

- `outputs/search_results.csv`: BFS/A* success, path/escape length, reward,
  expanded nodes, and planning time for every flood scenario.
- `outputs/rl_results.csv`: Random, Q-Learning, and PPO comparison metrics.
- `outputs/learning_curves.csv`: episode returns for Q-Learning/PPO.
- `outputs/models/`: saved Q tables and PPO policies.
- `outputs/plots/`: grid/path, flood progression, learning curves, reward and
  success comparisons, Q policy map, and animated A* flood escape GIF.

The Random-policy metrics are included in the result table. Learning curve
plots and policy/reward comparisons are created by the full run.

## Visual dashboard

Open `index.html` in a browser to explore the BFS/A* search benchmark. The
standalone dashboard includes scenario filters, search-efficiency and success
charts, a flood-timeline map, and the run-by-run results table. Its 30-second
agent demo replays the A* route for the normal-flood scenario, with controls to
pause, replay, and enter fullscreen presentation mode. Start a screen recording
after entering presentation mode and before selecting **Start 30 s demo** to
capture the full timed run. The dashboard has
no additional dependencies; its benchmark values are embedded from
`outputs/search_results.csv`.

## Report outline

1. **Abstract:** task, methods, headline success-rate and runtime results.
2. **Environment:** state/action space, wall layout, flood layers, rewards,
   horizon, and deterministic assumptions.
3. **Methods:** BFS/A* time-expanded planning; A* Manhattan heuristic;
   Q-Learning update and exploration; PPO implementation; random baseline.
4. **Experimental setup:** four flood speeds, basic/shaped rewards, seed,
   episode/timestep counts, hardware/software.
5. **Results:** success rate, escape time, path length, return, search
   expansions, computation time; include generated plots and CSV-derived table.
6. **Discussion:** shaping effects, speed robustness, optimality/information
   differences, and limitations (fixed map, known flood schedule for search).
7. **Conclusion:** key findings and one realistic extension.

## PPT outline

1. Title and 1-sentence problem.
2. Environment grid and flood progression.
3. State, actions, dynamics, and reward.
4. Search vs RL methods (include time-expanded search and Manhattan heuristic).
5. Experiment matrix (4 speeds × 2 rewards).
6. Results: success and reward plots.
7. Results: path/expansions/runtime and learning curves.
8. Takeaways, limitations, and demo.

## 30-second demonstration

1. (0–5 s) Show the grid with start, safe zone, walls, and flood source.
2. (5–12 s) Show `grid_bfs_astar.png`; point out that both planners account
   for the same forecast and A* expands fewer nodes when the result supports it.
3. (12–20 s) Play `flood_escape.gif`: the route and flood advance over time.
4. (20–27 s) Show `success_rate_comparison.png` or `learning_curves.png` and
   contrast basic vs shaped reward.
5. (27–30 s) State the main result and caveat: small fixed grid, not a fluid or
   real-world rescue simulator.
