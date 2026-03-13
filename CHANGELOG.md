# Changelog — AI Enemy Kit

## [1.0.0] — 2024

### Added
- EnemyAI: 8-state machine (Idle, Patrol, Investigate, Alert, Combat, TakeCover, Retreat, Dead)
- EnemyAI: 4 enemy types (Infantry, Heavy, Sniper, Scout, Grenadier)
- EnemyAI: 4 combat styles (Aggressive, Balanced, Defensive, Flanker)
- EnemyAI: Burst fire system with configurable duration and pause
- EnemyAI: Cover evaluation scoring (distance, LOS, protection angle)
- EnemyAI: Peek & shoot while in cover
- EnemyAI: Grenade throwing with static-target detection
- EnemyAI: Flanking behaviour for Scout/Flanker type
- EnemyAI: Squad ID system with alert propagation
- EnemyAI: Reinforcement calling with configurable delay
- EnemyAI: Retreat on low HP threshold
- EnemyAI: Stun API with auto-recovery
- EnemyAI: Full Gizmo visualization (ranges, LKP, squad radius)
- EnemyPerception: FOV cone with configurable angle and range
- EnemyPerception: Multi-height LOS check (3 rays: head, body, low)
- EnemyPerception: Peripheral detection zone
- EnemyPerception: Coroutine-based scan for performance
- EnemyPerception: Hearing system with SoundEmitter component
- EnemyPerception: Suppressed weapon hearing reduction
- EnemyPerception: Time-decay memory system per target
- EnemyPerception: Gizmo visualization (FOV cone, hearing, peripheral)
- EnemyHealth: Hit zone multipliers (head, body, limb)
- EnemyHealth: Armour reduction coefficient
- EnemyHealth: Health regen with configurable delay
- EnemyHealth: Optional ragdoll on death
- EnemyHealth: IDamageable implementation (FPS Kit compatible)
- EnemyWeapon: Hitscan raycast with damage falloff AnimationCurve
- EnemyWeapon: Burst firing, reload, magazine system
- EnemyWeapon: Moving spread multiplier
- EnemyWeapon: Suppressor support (audio + hearing reduction)
- EnemyWeapon: Muzzle flash prefab slot
- CoverPoint: Scored cover placement component with Gizmos
- SoundEmitter: Audio event bridge for FPS Kit weapon integration
- EnemyManager: Wave system with configurable wave data
- EnemyManager: NavMesh-based spawn system with min-distance check
- EnemyManager: Global alarm system
- EnemyManager: Active enemy tracking, kill counter
- EnemyManager: API: SpawnEnemy, AlertAllEnemies, KillAllEnemies, GetNearestEnemy
- AIEnemyKitBuilder: One-click demo arena with cover, patrol, spawn points
