# PowerShell Module Publishing

## Module Naming

The module name should always start with `PPG.`, followed by the distinctive name in CamelCase.

`PPG.ModuleName`

### Syntax Explanation

VMWare and Microsoft modules start with their respective company name, then using dot notation specify the product category.  The product and category is a bit too verbose for PPG needs.  The reasoning behind prefixing the module with the company name is so that the modules written by that company are easily identifiable.  Not only that but, if a function or cmdlet has a name collision, the module acts as a namespace when calling said function or cmdlet.

### Recommendations  

The module name should either be generic or specific to the product. For example, a module name could be PPG.Encryption, this would contain functions that have something to do with encryption.  Inversely a module could be named PPG.PowerShellService, because it contains all the function necessary to facilitate creating a Service via PowerShell.

## Function Naming

Naming functions correctly is important because it will allow users to inherently know which commands to type. When importing modules that include functions that are named with non-approved verbs, PowerShell will write to the warning stream that non-approved verbs are used.  Please reference the Microsoft article linked below for help choosing the correct verbs.

Approved Verbs for Windows PowerShell Commands

[https://docs.microsoft.com/en-us/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands?view=powershell-7.1](https://docs.microsoft.com/en-us/powershell/scripting/developer/cmdlet/approved-verbs-for-windows-powershell-commands?view=powershell-7.1)

### Function Naming Conflicts

To avoid naming conflicts, specify the module name when invoking functions. See the following syntax examples.

`PPG.ModuleName\New-FunctionName`

`PPG.ModuleName\Remove-FunctionName`

## Folder Structure

This is the folder structure for modules.

```folder-structure
- PPG.ModuleName\
    - functions\
        - New-FunctionName.ps1
        - Remove-FunctionName.ps1
    - PPG.ModuleName.psm1
    - PPG.ModuleName.psd1
```

## Install Location  

For manual installation of modules, copy them to the `C:\Program Files\WindowsPowerShell\Modules\` directory.

## Versioning

PowerShell modules should contain a manifest file denoted by the `.psd1` extension. There the Version number can be found and updated when new code is released.

### Syntax

Major.Minor.Build.Revision

#### Major

Assemblies with the same name but different major versions are not interchangeable. A higher version number might indicate a major rewrite of a product where backward compatibility cannot be assumed.

#### Minor

If the name and major version number on two assemblies are the same, but the minor version number is different, this indicates significant enhancement with the intention of backward compatibility. This higher minor version number might indicate a point release of a product or a fully backward-compatible new version of a product.

#### Build

A difference in build number represents a recompilation of the same source. Different build numbers might be used when the processor, platform, or compiler changes.

For purposes of PowerShell Modules, this can always be left at 0.

#### Revision

Assemblies with the same name, major, and minor version numbers but different revisions are intended to be fully interchangeable. A higher revision number might be used in a build that fixes a security hole in a previously released assembly.

#### Module Version Example

`1.3.0.2`

**1** means this is the first major release. 1.0.0.0 would have been the first version of the product.

**3** means this is the third minor release under major version 1.  

**0** is the build number for all releases

**2** means this is the second set of fixes to the minor release 3.

## Module Manifest

### Preface

PowerShell Module Manifests are not required but highly recommended. It describes the module and has the `.psd1` extension.

When importing a module from the filesystem, it should be imported by specifying the filename of the Module Manifest.

### Manifest Example

```powershell
@{
    RootModule      = 'PPG.HA.psm1'
    ModuleVersion   = '2.0.0.3'
    Author          = 'PIA'
    CompanyName     = 'PPG'
    Description     = '2 node minimum application level high availability (HA). Uses redis to store cluster state'
    RequiredModules = @(@{ModuleName = 'PPG.RedisClient'; ModuleVersion = '2.4.0.0'; })
}
```

### Manifest Basics

The naming and versioning of the manifest have already been covered.

An author is not necessary but if specified, it is best to specify your team name.

The description should be a brief description of what the module offers or accomplishes.

### Root Module

The root module is the filename of the `.PSM1` file.  The `.PSM1` file should either auto load all functions or contain all functions.  It is recommended that the former method is used, see below.

```powershell
$ModulePath = Split-Path $script:MyInvocation.MyCommand.Path
(Get-ChildItem "$($ModulePath)\functions" -Filter "*ps1").FullName | ForEach-Object {if ($_){. $_}}
```

### Depending on PowerShell Modules and DLLs

Module dependencies are crucial for reusing code. The module name and minimum version of the module can be specified.  When a module is imported that depends on another module, and that dependency is not yet imported, as long as it is in `$PSModulePath`, it will be automatically imported. If the dependency is not in `$PSModulePath`, then it must be imported manually before importing the dependent module.

If the required modules are managed externally you must specify them as an external module or the module publishing will fail.

#### Dependency Example

```powershell
@{
    # Other manifest details omitted
    RequiredModules = @( 
        @{ModuleName='PPG.PSThread'; ModuleVersion='1.0.0.1';}, 
        @{ModuleName='PPG.Encryption'; ModuleVersion='2.0.0.3';})
    PrivateData = @{
        PSData = @{
            ExternalModuleDependencies = @('PPG.PSThread','PPG.Encryption')
        }
    } 
}
```

### Nesting PowerShell Modules and DLLs

To nest a module or DLL, that means it will be loaded as part of the module that is being imported.  The nested module is stored along with the module that is being imported.  Unlike a required module, where it is a separate module that is just a dependency.

#### Nested Example

```powershell
@{
    # Other manifest details omitted
    NestedModules = @('StackExchange.Redis.dll')`
}
```

### Hiding and Exposing Functions and or Cmdlets

If only certain functions or cmdlets should be exposed to the user, and the rest are only used internally by the module, use the export keys.

### Hiding and Exposing Example

```powershell
@{
    # Other manifest details omitted
    FunctionsToExport = @( 
        'Add-SomeThing', 
        'Remove-SomeThing'
    ) 
    CmdletsToExport = @( 
        'Start-SomeThing', 
        'Stop-SomeThing'
    )
}
```

### Packaging other Files

If supporting files are stored in the module folder, then use the FileList key to specify them in an array.  This feature inert, the purpose is to keep an inventory of supporting files.

### Packaging Example

```powershell
@{
    # Other manifest details omitted
    FileList = @( 
        "DeployBitnami.json", 
        "redis-trib.rb"
    )
}
```

## References

[http://msdn.microsoft.com/en-us/library/system.version.aspx](http://msdn.microsoft.com/en-us/library/system.version.aspx)

[https://softwareengineering.stackexchange.com/questions/24987/what-exactly-is-the-build-number-in-major-minor-buildnumber-revision](https://softwareengineering.stackexchange.com/questions/24987/what-exactly-is-the-build-number-in-major-minor-buildnumber-revision)
