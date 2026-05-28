# SPUD

Save game and streaming level persistence for Unreal Engine 5.

## Capabilities

| Capability | Description |
|------------|-------------|
| **ISpudObject Interface** | Marker interface to opt classes into persistence |
| **SaveGame Property Flag** | Mark properties with `UPROPERTY(SaveGame)` for persistence |
| **Automatic State Saving** | Transform, controller rotation, physics velocities saved automatically |
| **Runtime Spawned Actors** | Re-spawned on load with `SpudGuid` property |
| **Streaming Support** | `USpudSubsystem` methods for streamed level management |
| **Custom Data Callbacks** | `ISpudObjectCallback` for pre/post save/restore hooks |

## Key Classes

| Class | Purpose |
|-------|---------|
| `USpudSubsystem` | Main interface for save/load operations |
| `ISpudObject` | Marker interface for persistent objects |
| `ISpudObjectCallback` | Save/restore lifecycle callbacks |
| `ISpudRoamingActor` | For actors moving between World Partition cells |
| `USpudState` | Internal state representation |
| `USpudStreamingVolume` | Streaming volume for SPUD-aware loading |

## Key Structs & Interfaces

| Name | Type | Purpose |
|------|------|---------|
| `ESpudRespawnMode` | Enum | Controls respawn behavior |
| `ISpudObject` | Interface | `GetSpudRespawnMode()`, `ShouldSkip()`, `OverrideName()` |
| `ISpudObjectCallback` | Interface | `SpudPreStore()`, `SpudPostRestore_Implementation()` |
| `ISpudRoamingActor` | Interface | Marker for roaming actor tracking |

## Common Pitfalls

- Not saving levels before PIE — unsynced levels cause state issues
- Forgetting `SaveGame` flag on properties — they won't be persisted
- Marking `SpudGuid` as `SaveGame` — it's metadata, not state
- Not implementing `OverrideName()` for framework actors — breaks identity
- Direct level streaming calls — use `USpudSubsystem` methods instead
- Missing `StructUtils` plugin — required for serialization

See `.claude/patterns.md` for implementation patterns.

## Integration Points

- Used by: SUQS (quest persistence), SUQSFlow, HorrorFeatures
- See `.claude/plugin-integration.md` for cross-plugin dependency matrix

## Human Review Required

- Changes to `ISpudObject` or `ISpudObjectCallback` interfaces
- Changes to `USpudSubsystem` public API or save file format
- See `.claude/human-review-checklist.md` for full list