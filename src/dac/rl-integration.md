# Godot RL Agents Integration for Autonomous Driving

**Author:** Mikayla Lee

## Overview
The Godot RL Agents integration enables reinforcement learning agents to train autonomous driving policies on the AER simulator. This system bridges the Godot physics simulator with RL training libraries, allowing any standard AI RL agents to learn driving behaviors in a virtual environment.

The integration serves as the interface layer between:
- Godot car physics simulator
- Stable-Baselines3 PPO training algorithms
- Future autonomous driving control systems

## System Architecture
The RL training pipeline consists of:
1. **GodotEnv Wrapper** - Connects RL agents to the Godot simulator
2. **Action Space** - Defines steering and throttle controls (2D continuous space)
3. **Observation Space** - Captures car telemetry (position, velocity, speed)
4. **PPO Training Model** - Implements Proximal Policy Optimization for policy learning
5. **Reset/Step Functions** - Handles episode management and action execution

The system uses the Godot RL Agents library to establish direct communication between the Python training environment and the Godot game engine, eliminating the need for manual WebSocket implementation.

## Implementation Details

### Environment Configuration
```python
class GodotCarEnv(GodotEnv):
    def __init__(self, env_path=None, port=11008, show_window=True, seed=0):
        super().__init__(
            env_path=env_path,
            port=port,
            show_window=show_window,
            seed=seed
        )
```

The environment wrapper inherits from `GodotEnv` and manages:
- Connection to Godot executable on port 11008
- Visual rendering control for training monitoring

### Action Space
Actions are represented as a 2D continuous vector `[steering, throttle]`:

| Control | Range | Mapping |
|---------|-------|---------|
| Steering | -1.0 to 1.0 | -20° to +20° |
| Throttle | -1.0 to 1.0 | Full brake to full gas |

### Observation Space
The agent receives 7D telemetry data per timestep:
- Position (x, y, z) - 3D coordinates in world space
- Velocity (vx, vy, vz) - 3D velocity vector
- Speed - Scalar magnitude of velocity

### Training Pipeline
The system uses Proximal Policy Optimization (PPO) with the following arameters:

| Parameter | Value | Purpose |
|-----------|-------|---------|
| Learning Rate | 3e-4 | Gradient descent step size |
| Steps per Update | 2048 | Experience collection before update |
| Batch Size | 64 | Mini-batch size for optimization |
| Epochs | 10 | Training iterations per update |
| Gamma | 0.99 | Discount factor for future rewards |

## Integration Workflow

The RL training loop operates as follows:

1. **Start:**  
   `reset()` resets car to starting position → returns initial observation

2. **Action Execution:**  
   Agent predicts action based on current observation → `step(action)` sends controls to Godot

3. **Simulation Update:**  
   Godot applies physics, updates car state → returns new observation and reward

4. **Learning:**  
   PPO updates policy based on collected experience → repeat until convergence

This cycle runs for 100,000 timesteps during initial training, with the trained model saved for deployment.

## Current Status & Next Steps

**Completed:**
- Researched and integrated Godot RL Agents library
- Implemented GodotEnv wrapper with action/observation spaces
- Configured PPO training pipeline with Stable-Baselines3
- Established reset/step function architecture

**In Progress:**
- Creating basic test simulator in Godot to validate integration
- Working with other daq members to fully integrate RL wrapper with car physics simulation

**Future Work:**
- Implement reward shaping for racing-specific behaviors (staying on track, lap time optimization)
- Add termination conditions (crash detection, lap completion)

## Dependencies
```bash
pip install godot-rl stable-baselines3
```

## References
- [Godot RL Agents GitHub](https://github.com/edbeeching/godot_rl_agents)
- [Stable-Baselines3 Documentation](https://stable-baselines3.readthedocs.io/)
- [PPO Algorithm Guide](https://stable-baselines3.readthedocs.io/en/master/modules/ppo.html)
