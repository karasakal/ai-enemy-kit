# 🤖 AI Enemy Kit — Professional Enemy AI System for Unity

[![Unity](https://img.shields.io/badge/Unity-2021.3%2B-black?logo=unity)](https://unity.com)
[![License](https://img.shields.io/badge/License-Commercial-red)](#license)
[![Asset Store](https://img.shields.io/badge/Asset%20Store-$19.99-orange)](https://assetstore.unity.com)
[![Publisher](https://img.shields.io/badge/Publisher-Haner%20Games-purple)](https://hanergames.com.tr)
[![FPS Kit](https://img.shields.io/badge/FPS%20Kit-Compatible-brightgreen)](https://hanergames.com.tr)

> Production-ready enemy AI system for Unity. NavMesh-based movement, 8-state machine, FOV + hearing perception, cover system, squad communication, wave spawner — fully compatible with FPS Kit. Zero third-party dependencies.

---

## 🧠 State Machine

```
                    ┌─────────┐
              ┌────▶│  IDLE   │◀────┐
              │     └────┬────┘     │
              │          │ timeout  │
              │     ┌────▼────┐     │
              │     │ PATROL  │     │ all enemies dead
              │     └────┬────┘     │
              │          │ sound    │
              │     ┌────▼──────┐   │
              │     │INVESTIGATE│   │
              │     └────┬──────┘   │
              │          │ seen     │
              │     ┌────▼────┐     │
   squad  ───▶│     │  ALERT  │     │
   alert       │     └────┬────┘     │
              │          │ react    │
              │     ┌────▼────┐     │
              │  ┌─▶│ COMBAT  │─────┤
              │  │  └────┬────┘     │
              │  │       │ damage   │
              │  │  ┌────▼──────┐   │
              │  └──│ TAKCOVER  │   │
              │     └────┬──────┘   │
              │          │ low HP   │
              │     ┌────▼────┐     │
              │     │ RETREAT │     │
              │     └────┬────┘     │
              │          │ death    │
              │     ┌────▼────┐     │
              └─────│  DEAD   │─────┘
                    └─────────┘
```

---

## ✨ Features at a Glance

### 🧠 State Machine (8 States)
- **Idle** — standing, waiting
- **Patrol** — waypoint loop or random wander
- **Investigate** — moves toward last heard sound
- **Alert** — runs to last known position, scans
- **Combat** — burst fire, positioning, cover evaluation
- **TakeCover** — seeks best cover point, peeks & shoots
- **Retreat** — withdraws when HP falls below threshold
- **Dead** — plays death animation, optional ragdoll

### 👁 Perception System
- **FOV cone** — configurable angle (20°–360°) and range
- **Multi-height LOS** — 3-ray check (head, body, low) for accurate detection
- **Peripheral zone** — short-range always-detect regardless of angle
- **Hearing** — radius-based, source-driven via `SoundEmitter` component
- **Suppressed hearing** — silenced weapons heard at configurable reduced range
- **Memory** — independently tracks multiple targets with time-decay
- **Crouch visibility** — enemy sees crouching players at reduced range
- **Performance** — coroutine-based scan rate (configurable per second)

### 🛡 Cover System
- `CoverPoint` component — place anywhere in scene
- **Smart selection** — scored by distance, LOS to threat, protection angle
- **Multi-occupant** — optionally allow multiple enemies per cover
- **Peek & shoot** — enemy leans out at intervals while in cover
- **Availability** — occupied cover excluded from selection

### 🎯 Combat Behaviours
- **Burst fire** — configurable burst duration and pause interval
- **Accuracy** — per-enemy 0–1 accuracy drives shot spread
- **Moving spread** — wider shots when enemy is running
- **Grenade** — throw grenade when target stationary, range-checked
- **Flanking** — flanker-type enemies attempt perpendicular approach
- **Melee** — triggered at close range
- **Retreat threshold** — configurable HP% triggers tactical withdrawal

### 🗣 Squad System
- **Alert propagation** — spotted enemy alerts nearby squad members
- **Squad ID** — group enemies by ID; cross-squad alerts don't occur
- **Global alarm** — optional manager-level alert broadcasts to all enemies
- **Reinforcement call** — delayed reinforcement request after engagement starts

### 🌊 Wave & Spawn System (`EnemyManager`)
- Configurable wave data (count, prefab, spawn interval)
- Min spawn distance from player
- Max concurrent enemies cap
- All enemies registration & tracking
- Events: `OnWaveStarted`, `OnWaveCleared`, `OnAllWavesCleared`, `OnEnemyKilled`

### 🔌 FPS Kit Integration
- Uses `IDamageable` interface — FPS Kit bullets damage enemies out of the box
- `SoundEmitter` on weapon → enemy `EnemyPerception.HearSound()` — shot detection
- Shared `IDamageable` namespace — zero bridge code required

---

## 🚀 Quick Start

### 1. Import
Copy `Assets/AIEnemyKit/` into your Unity project.  
*(If using FPS Kit: both `Assets/CarKit/` and `Assets/AIEnemyKit/` in the same project.)*

### 2. Build Demo Scene
```
AI Enemy Kit → 🤖 Build Demo Scene
```

### 3. Bake NavMesh
```
Window → AI → Navigation → Bake
```

### 4. Play ▶

---

## 🎮 Enemy Types & Recommended Settings

| Type | HP | Accuracy | Combat Style | Grenade | Flank |
|------|----|----------|--------------|---------|-------|
| Infantry | 100 | 0.70 | Balanced | ✗ | ✗ |
| Heavy | 250 | 0.55 | Defensive | ✗ | ✗ |
| Sniper | 75 | 0.95 | Defensive | ✗ | ✗ |
| Scout | 80 | 0.65 | Aggressive | ✗ | ✓ |
| Grenadier | 100 | 0.60 | Balanced | ✓ | ✗ |

---

## 📋 API Reference

### EnemyAI

```csharp
// State
AIState  CurrentState      // Idle, Patrol, Investigate, Alert, Combat, TakeCover, Retreat, Dead
bool     HasTarget
bool     IsDead
bool     IsInCover
bool     IsShooting
float    DistanceToTarget
Vector3  LastKnownPos
string   squadID

// Methods
void  SetState(AIState state)
void  ForceTarget(Transform t)       // Immediately engage a target
void  ClearTarget()                  // Drop target, return to Alert
void  Stun(float duration)           // Temporary stun
void  ReceiveSquadAlert(Transform t, Vector3 lastKnown)
void  SetPatrolPoints(Transform[] points)

// Events
event Action<AIState, AIState>  OnStateChanged     // (old, new)
event Action<Transform>         OnTargetAcquired
event Action                    OnTargetLost
event Action                    OnDeath
event Action                    OnAlert
event Action<Vector3>           OnGrenadeThrownAt
```

### EnemyPerception

```csharp
bool      CanSeeTarget
Transform PrimaryTarget
bool      IsAlert

// Direct sound notification
void HearSound(Vector3 worldPos, float loudness = 1f)

// Memory queries
bool     RemembersTarget(Transform t)
Vector3? GetLastKnownPosition(Transform t)

// Events
event Action<Transform> OnTargetSpotted
event Action<Transform> OnTargetLostSight
event Action<Vector3>   OnSoundHeard
event Action<Transform> OnNewThreatDetected
```

### EnemyHealth

```csharp
float HealthNormalized   // 0–1
bool  IsDead

void TakeDamage(float dmg, Vector3 hitPoint, Vector3 hitNormal, string hitTag)
void Heal(float amount)
void SetHealth(float value)

event Action<float, Vector3> OnDamaged    // (damage, hitPoint)
event Action                 OnDeath
event Action<float>          OnHealed
```

### EnemyManager

```csharp
int   ActiveEnemyCount
int   TotalKilled
int   CurrentWave
bool  IsGlobalAlarm

EnemyAI          SpawnEnemy(int prefabIndex = 0, int spawnPointIndex = -1)
void             AlertAllEnemies(Transform target, Vector3 lastKnown)
void             KillAllEnemies()
List<EnemyAI>   GetEnemiesInRadius(Vector3 center, float radius)
EnemyAI          GetNearestEnemy(Vector3 pos)

event Action<int>  OnEnemyKilled
event Action<int>  OnWaveStarted
event Action<int>  OnWaveCleared
event Action       OnAllWavesCleared
event Action       OnGlobalAlarmRaised
```

---

## 🗂 Package Structure

```
Assets/AIEnemyKit/
├── Scripts/
│   ├── AI/
│   │   ├── EnemyAI.cs             # Core state machine
│   │   ├── EnemyComponents.cs     # Health, Weapon, CoverPoint, SoundEmitter
│   │   └── EnemyManager.cs        # Spawner, waves, squad coordinator
│   └── Perception/
│       └── EnemyPerception.cs     # FOV, hearing, memory
├── Editor/
│   └── AIEnemyKitBuilder.cs       # One-click demo scene builder
└── Documentation/
    └── README.md
```

---

## 🏷 Scene Setup (Manual)

```
Enemy_01                    [NavMeshAgent, EnemyAI, EnemyPerception,
                             EnemyHealth, EnemyWeapon, Animator]
├── EyeTransform            [Perception eye position]
└── MuzzlePoint             [Weapon fire origin]

CoverPoint_01               [CoverPoint]  ← place around scene
CoverPoint_02               [CoverPoint]

SpawnPoint_01               [Transform]   ← empty GameObjects
SpawnPoint_02               [Transform]

EnemyManager                [EnemyManager]
```

---

## 🔗 FPS Kit Integration Guide

See [FPS_Kit_Integration.md](./FPS_Kit_Integration.md) for the full guide.

**TL;DR:**
```csharp
// EnemyHealth already implements IDamageable
// FPS Kit's WeaponController fires → hits enemy → TakeDamage() called automatically

// Sound: add SoundEmitter to WeaponController's fire point
soundEmitter.IsSuppressed = weaponController.IsSilenced;
soundEmitter.Emit();  // call this on every shot
```

---

## 📦 Technical Specs

| Item | Value |
|------|-------|
| Unity | 2021.3 LTS+ |
| Render Pipeline | Built-in · URP · HDRP |
| NavMesh | Unity built-in AI |
| Dependencies | **None** (FPS Kit optional) |
| Scripts | 4 C# files |
| Lines of code | ~2,350 |
| AI States | 8 |
| Perception | Vision + Hearing + Memory |
| Platforms | Win · macOS · Linux · iOS · Android |

---

## 🛒 Purchase

**[Unity Asset Store →](https://assetstore.unity.com)**

Source code is distributed exclusively through the Unity Asset Store.
This repository contains documentation and issue tracking only.

---

## 📬 Support

- **Bug reports & feature requests:** [Open an Issue](https://github.com/karasakal/ai-enemy-kit/issues)
- **Documentation:** Included in package + this README
- **FPS Kit integration:** [FPS_Kit_Integration.md](./FPS_Kit_Integration.md)
- **Publisher:** [hanergames.com.tr](https://hanergames.com.tr)

---

## 📄 License

AI Enemy Kit is a **commercial asset**. Source code available only to Asset Store purchasers.

- ✅ Unlimited personal and commercial projects
- ✅ Modify for your own project needs
- ❌ Redistribute, resell, or include in other packages
- ❌ Share source code publicly

© 2026 Haner Games — hanergames.com.tr
