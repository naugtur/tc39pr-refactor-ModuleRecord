# ModuleRecord refactoring diagrams

The purpose of this document is to ensure complete understanding of changes proposed in README.md by an attempt to reproduce them in a different medium.


## Before


> `☑️` means the field has a known place in "After",  
> `❎` means it does not.

```mermaid
classDiagram
direction LR
    ModuleRecord <|-- CyclicModuleRecord
    CyclicModuleRecord <|-- SourceTextModuleRecord

    CyclicModuleRecord

    class ScriptRecord{
        ☑️ Realm: RealmRecord
        ☑️ ECMAScriptCode
        ☑️ HostDefined
    }
    class ModuleRecord{
        <<abstract>>
        ☑️ Realm: RealmRecord
        ☑️ Environment: ModuleEnvironmentRecord
        ☑️ Namespace
        ☑️ HostDefined

        - ❎ GetExportedNames()
        - ❎ ResolveExport()
        - ❎ Link()
        - ❎ Evaluate()
    }
    class CyclicModuleRecord{
        <<abstract>>
        ☑️ Status
        ☑️ EvaluationError
        ☑️ DFSIndex
        ☑️ DFSAncestorIndex
        ☑️ RequestedModules: List of String
        ❎ CycleRoot: CyclicModuleRecord
        ❎ HasTLA
        ❎ AsyncEvaluation
        ❎ TopLevelCapability
        ❎ AsyncParentModules
        ❎ PendingAsyncDependencies

        - ❎ InitializeEnvironment() 
        - ❎ ExecuteModule()
    }
    class SourceTextModuleRecord{
        ☑️ ECMAScriptCode
        ☑️ Context
        ❎ ImportMeta
        ☑️ ImportEntries
        ☑️ LocalExportEntries
        ☑️ IndirectExportEntries
        ☑️ StarExportEntries
    }

    class RealmRecord{
        ☑️ Intrinsics
        ☑️ GlobalObject
        ☑️ GlobalEnv: GlobalEnvironmentRecord
        ☑️ TemplateMap
        ☑️ HostDefined
    }


```


## After

```mermaid
classDiagram
direction LR
    StaticModuleRecord <|-- CyclicStaticModuleRecord
    CyclicStaticModuleRecord <|-- SourceTextStaticModuleRecord

    class ScriptInstance{
        EvalRecord: EvalRecord [was RealmRecord]
        ECMAScriptCode
        ?? Environment
        HostDefined
    }
    class StaticModuleRecord{
        <<abstract>>
    }
    class CyclicStaticModuleRecord{
        <<abstract>>
        RequestedModules: List of String
    }
    class SourceTextStaticModuleRecord{
        ECMAScriptCode
        ImportEntries
        LocalExportEntries
        IndirectExportEntries
        StarExportEntries
    }

  
    class ModuleInstance{
        StaticModuleRecord: StaticModuleRecord
        EvalRecord: EvalRecord [was RealmRecord]
        Environment
        Namespace
        HostDefined
    }

    %% was RealmRecord
    class EvalRecord {
        Intrinsics
        GlobalObject
        GlobalEnv: GlobalEnvironmentRecord
        TemplateMap
        HostDefined

        - ResolveImportedModule()
        - ImportModuleDynamically()
        - ResolveImportMeta()
    }

    class ModuleInitialization{
        ModuleInstance: ModuleInstance
        Status
        EvaluationError
        DFSIndex
        DFSAncestorIndex
        Context
    }


```

# Notes

## Open questions

1. Why is CyclicStaticModuleRecord not abstract in the refactoring writeup? I made it abstract, seems like it simply wasn't stated
2. Is `ResolveImportMeta()` meant to replace `SourceTextModuleRecord.ImportMeta` ?
3. Was `Environment` originally not present in Script Record, because it was always using `Realm.GlobalEnv`? Does it change now?


## TODO
- find `referrer` and put it on the map
- figure out where fields marked with `-` in "Before" go
- (optionally) draw out relationships