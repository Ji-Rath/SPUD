# AGENTS.md

This file provides guidance to agents when working with the SPUD plugin.

## 1. Plugin Overview

**SPUD** (Steve's Persistent Unreal Data) is a save game and streaming level persistence solution for Unreal Engine 5. It provides two main features:

1. **Save / Load game state easily** - Mark actors as persistent and save their state
2. **Streamed levels retain their state** - Level state persists as levels unload/reload without a save needed

### Key Capabilities

| Capability | Description |
|------------|-------------|
| **ISpudObject Interface** | Marker interface to opt classes into persistence |
| **SaveGame Property Flag** | Use `UPROPERTY(SaveGame)` to mark properties for persistence |
| **Automatic State Saving** | Transform, controller rotation, physics velocities saved automatically |
| **Runtime Spawned Actors** | Re-spawned on load with `SpudGuid` property |
| **Gameplay Framework Actors** | Pawns/Characters/PlayerState/GameState/GameMode support via `OverrideName()` |
| **Streaming Support** | `USpudSubsystem` methods for streamed level management |
| **Custom Data Callbacks** | `ISpudObjectCallback` for pre/post save/restore hooks |

## 2. Runtime Requirements

| Environment | Minimum Version | Recommended Version |
|-------------|-----------------|---------------------|
| Unreal Engine | 5.0 | 5.7 |
| Windows | 10 | 11 |

## 3. Dependencies

### Build Dependencies (Build.cs)

| Module | Type | Required |
|--------|------|----------|
| Core | Public | Yes |
| CoreUObject | Public | Yes |
| Engine | Public | Yes |
| StructUtils | Public | Yes (plugin dependency) |

### Plugin Dependencies (uplugin)

| Plugin | Purpose |
|--------|---------|
| **StructUtils** | Property serialization utilities |

## 4. Module & Loading

| Property | Value |
|----------|-------|
| Runtime module | `SPUD` |
| Module type | `Runtime` |
| Loading phase | `Default` |
| Editor module | `SPUDEditor` (Editor) |
| Test module | `SPUDTest` (DeveloperTool) |

## 5. Key Classes

| Class | Type | Purpose |
|-------|------|---------|
| `USpudSubsystem` | `UGameInstanceSubsystem` | Main interface for save/load operations |
| `ISpudObject` | `UInterface` | Marker interface for persistent objects |
| `ISpudObjectCallback` | `UInterface` | Interface for save/restore lifecycle callbacks |
| `ISpudRoamingActor` | `UInterface` | For actors that move between World Partition cells |
| `USpudState` | `UObject` | Internal state representation |
| `USpudStreamingVolume` | `AVolume` | Streaming volume for SPUD-aware level loading |

## 6. Key Structs

| Struct | Blueprint Type | Purpose |
|--------|---------------|---------|
| `ESpudRespawnMode` | Yes | Controls respawn behavior for runtime objects |

## 7. Key Interfaces

| Interface | Methods |
|-----------|---------|
| `ISpudObject` | `GetSpudRespawnMode()`, `ShouldSkipRestoreTransform()`, `ShouldSkipRestoreVelocity()`, `OverrideName()`, `ShouldSkip()` |
| `ISpudObjectCallback` | `SpudPreStore()`, `SpudStoreCustomData()`, `SpudPostStore()`, `SpudPreRestore()`, `SpudRestoreCustomData()`, `SpudPostRestore()` |
| `ISpudRoamingActor` | Marker interface for roaming actor tracking |

## 8. Project Structure

```
SPUD/
├── Source/
│   ├── SPUD/                    # Runtime module
│   │   ├── Public/
│   │   │   ├── ISpudObject.h    # Main persistence interface
│   │   │   ├── SpudSubsystem.h  # Main subsystem
│   │   │   ├── SpudState.h      # State representation
│   │   │   └── SpudData.h       # Data structures
│   │   └── Private/
│   ├── SPUDEditor/              # Editor module
│   └── SPUDTest/                # Test module
├── doc/
│   ├── faq.md
│   ├── levelstreaming.md
│   ├── props.md
│   └── tech.md
└── Config/
```

## 9. Usage Patterns

### Implementing ISpudObject (C++)

```cpp
class AMyActor : public AActor, public ISpudObject
{
    // No methods required - it's a marker interface
    // Properties marked with SaveGame will be persisted
};
```

### Marking Properties for Save

```cpp
UPROPERTY(SaveGame)
int32 MySavedInt;

UPROPERTY(SaveGame)
FString MySavedString;
```

### Runtime Spawned Actors

```cpp
// Add SpudGuid property to runtime-spawned actors
UPROPERTY()
FGuid SpudGuid;
// SPUD will auto-generate if empty
```

### Gameplay Framework Actors

```cpp
FString AMyPlayerState::OverrideName_Implementation() const
{
    static const FString Name("PlayerState");
    return Name;
}
```

## 10. Non-Obvious Code Patterns

- **SaveGame flag required**: Only properties marked with `UPROPERTY(SaveGame)` are persisted. The interface alone doesn't save properties.
- **SpudGuid for runtime actors**: Runtime-spawned actors need a `SpudGuid` property for unique identification. Do NOT mark it as SaveGame.
- **OverrideName for framework actors**: Pawns, Characters, PlayerState, GameState, GameMode need `OverrideName()` to maintain identity across save/load.
- **SPUD + PIE requirement**: Always save all levels before entering Play-In-Editor. Enable "Save All Levels On Play In Editor" in SPUD settings.
- **Streaming through USpudSubsystem**: Use `USpudSubsystem` methods for level streaming to maintain persistence state.

## 11. Common Pitfalls for New Contributors

1. **Not saving levels before PIE**: Unsaved levels cause mis-categorization of level objects. Always save or enable auto-save in SPUD settings.
2. **Forgetting SaveGame flag**: Properties won't be saved without `UPROPERTY(SaveGame)`.
3. **Marking SpudGuid as SaveGame**: The GUID is metadata, not save state. Don't mark it as SaveGame.
4. **Not implementing OverrideName**: Framework actors get new names on each creation, breaking save identity.
5. **Direct level streaming calls**: Use `USpudSubsystem` streaming methods instead of direct `UGameplayStatics` calls.
6. **Missing StructUtils dependency**: Required for property serialization.

## 12. Build & Test Commands

### Build Editor Target
```bash
"<EnginePath>/Engine/Build/BatchFiles/Build.bat" SPUDEditor Win64 Development -Project="<ProjectPath>/YourProject.uproject" -WaitMutex -FromMSBuild
```

### Run SPUD Tests
```bash
RunUAT BuildCookRun -project=YourProject.uproject -noPMT -skipbuild -skiplog -clientconfig=Development -serverconfig=Development -runautomationtest -test=SPUDTest
```

## 13. Documentation

- [FAQ](doc/faq.md)
- [Level Streaming](doc/levelstreaming.md)
- [Properties](doc/props.md)
- [Technical Details](doc/tech.md)

## 14. Human Review Required Before Implementing

- Changes to `ISpudObject` interface
- Changes to `ISpudObjectCallback` interface
- Changes to `USpudSubsystem` public API
- Changes to save file format
- Changes to `SpudData.h` structures