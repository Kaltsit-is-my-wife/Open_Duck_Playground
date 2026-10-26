# Project Memory and Agent Agreement

## Memory policy

- This file is the single shared memory document for every agent working in this repository.
- All durable project knowledge, decisions, conventions, and task handoff notes must be recorded here.
- Do not create or use separate agent-specific memory files for this project.
- Keep this document concise and update it only when the shared project knowledge changes.

## Project overview

- The project is `Open_Duck_Playground`, a Python project for training and evaluating the Open Duck Mini V2 robot.
- The main stack is JAX, MuJoCo/MJX, Brax PPO, and `mujoco_playground`.
- Dependencies and Python packaging are defined in [`pyproject.toml`](./pyproject.toml); the project requires Python 3.11 or newer.
- The top-level [`README.md`](./README.md) documents installation, training, inference, and the expected project structure.
- The end-to-end purpose is: define a MuJoCo robot and terrain, train a PPO policy in an MJX environment, export the policy to ONNX, and run interactive MuJoCo inference.

## Current project status

- The repository currently contains the Open Duck Mini V2 implementation and shared training utilities; no other robot implementation is documented or configured.
- The root [`docs/`](./docs/) directory has been created as a future documentation location and is intentionally empty.
- This [`CLAUDE.md`](./CLAUDE.md) is the shared memory and handoff document for all agents. Update it when durable project knowledge or implementation decisions change.
- No source-code behavior changes have been made as part of the documentation setup. The current task history only added [`CLAUDE.md`](./CLAUDE.md) and the empty [`docs/`](./docs/) directory.
- Runtime outputs such as checkpoints, ONNX files, TensorBoard logs, and JAX caches are not part of the source documentation and should not be added here.

## End-to-end workflow

1. [`constants.py`](./playground/open_duck_mini_v2/constants.py) maps a task name to a MuJoCo scene XML.
2. [`base.py`](./playground/open_duck_mini_v2/base.py) loads the XML and assets, creates the MuJoCo/MJX model, and builds actuator, joint, floating-base, and backlash-joint mappings.
3. [`joystick.py`](./playground/open_duck_mini_v2/joystick.py) or [`standing.py`](./playground/open_duck_mini_v2/standing.py) creates the RL environment. Each step consumes observations, applies policy actions, advances simulation, and computes task rewards.
4. [`playground/open_duck_mini_v2/runner.py`](./playground/open_duck_mini_v2/runner.py) selects the environment and task, then delegates PPO training to [`playground/common/runner.py`](./playground/common/runner.py).
5. The shared runner writes TensorBoard metrics, saves Orbax checkpoints, and exports ONNX policies.
6. [`mujoco_infer.py`](./playground/open_duck_mini_v2/mujoco_infer.py) loads an ONNX policy and constructs inference observations from MuJoCo sensors, actuator joint state, previous actions, motor targets, contacts, and (for locomotion) imitation phase. Keyboard input controls locomotion and head commands.

## Environments and training behavior

- The `joystick` environment tracks commanded linear and angular velocity. Its current source settings enable imitation reward and motor-speed limits.
- The `standing` environment trains posture stability and standing-related rewards. Its current source settings disable imitation reward.
- The current task names are `flat_terrain`, `rough_terrain`, `flat_terrain_backlash`, and `rough_terrain_backlash`; confirm the corresponding XML files before adding or renaming tasks.
- Imitation-reward paths require [`polynomial_coefficients.pkl`](./playground/open_duck_mini_v2/data/polynomial_coefficients.pkl). Do not remove or relocate this data without updating the loading paths.
- Training is normally launched with `uv run playground/open_duck_mini_v2/runner.py`; useful options include `--env`, `--task`, `--num_timesteps`, `--output_dir`, and `--restore_checkpoint_path`.
- Interactive inference is currently specific to Open Duck Mini V2 and expects a compatible ONNX policy, model path, and reference data as configured by the script.

## Handoff guidance for future agents

- Read this file first, then inspect the relevant module and its XML scene before changing behavior.
- Keep training-time observations, action scaling, joint ordering, motor limits, and inference-time observations synchronized. Changes to one side usually require checking the other side.
- When changing robot joints, sensors, actuators, or body names, update both Python constants/helpers and the relevant files under [`playground/open_duck_mini_v2/xmls/`](./playground/open_duck_mini_v2/xmls/).
- Prefer existing helpers in [`playground/common/`](./playground/common/) and the existing environment patterns over duplicated logic.
- Use the smallest relevant validation command. For training or inference changes, first verify imports/configuration where possible; full PPO training is expensive and requires the project dependencies and suitable hardware.

## Open Duck Mini V2

- Robot-specific code is in [`playground/open_duck_mini_v2/`](./playground/open_duck_mini_v2/).
- [`base.py`](./playground/open_duck_mini_v2/base.py) loads MuJoCo XML assets and builds the MJX environment metadata, including actuator, joint, floating-base, and backlash-joint mappings.
- [`constants.py`](./playground/open_duck_mini_v2/constants.py) maps task names to XML scenes and defines robot joint, foot, body, and sensor names.
- [`joystick.py`](./playground/open_duck_mini_v2/joystick.py) defines the locomotion task, command ranges, imitation reward usage, motor speed limits, observations, reset logic, and rewards.
- [`standing.py`](./playground/open_duck_mini_v2/standing.py) defines the standing task and its reward configuration.
- [`runner.py`](./playground/open_duck_mini_v2/runner.py) selects the joystick or standing environment and delegates training to the shared runner.
- [`mujoco_infer.py`](./playground/open_duck_mini_v2/mujoco_infer.py) runs interactive MuJoCo inference from an ONNX policy; keyboard commands control locomotion or the head.
- [`mujoco_infer_base.py`](./playground/open_duck_mini_v2/mujoco_infer_base.py) contains shared MuJoCo inference/model-access helpers.
- [`xmls/`](./playground/open_duck_mini_v2/xmls/) contains the robot model, terrain scenes, sensors, joint properties, configuration, and mesh assets.
- [`data/polynomial_coefficients.pkl`](./playground/open_duck_mini_v2/data/polynomial_coefficients.pkl) contains reference-motion polynomial coefficients used by imitation-reward paths.

## Existing conventions

- Shared utilities and training infrastructure live under [`playground/common/`](./playground/common/).
- Training is normally launched with `uv run playground/open_duck_mini_v2/runner.py`.
- The task names currently mapped by `constants.task_to_xml` are `flat_terrain`, `rough_terrain`, `flat_terrain_backlash`, and `rough_terrain_backlash`.
- Training checkpoints, ONNX exports, TensorBoard logs, and JAX cache files are runtime outputs and should not be treated as source documentation.

## Documentation layout

- The root [`docs/`](./docs/) directory is reserved for future project documentation.
- It is intentionally empty for now.
