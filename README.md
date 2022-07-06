# Introduction

A template for often used features of Azure Pipelines.

## Installation

No installation required.

## Configuration

Configure the pipeline in the yaml file. 

## Usage

Azure Pipelines are organized top down in Stages -> Jobs -> Steps. 

* Stages can be optional: when running the pipeline, a user can chose which stages to run. They are also used for grouping Jobs.

* Jobs are principally used for grouping Steps. 

* Steps are the smallest units in Azure Pipelines, they are either 
	* Tasks: predefined actions on Azure Pipelines that can be configured with additional parameters
	* Scripts: user-defined actions

e.g.

Install NodeJS v10

```
- task: NodeTool@0
      inputs:
        versionSpec: '10.x'
      displayName: 'Install NodeJS'
```

Install NodeJS Dependencies (e.g. "smallest" module)

```
- script: npm i smallest
      workingDirectory: '$(Build.SourcesDirectory)/$(myDirectory)'
      displayName: 'Install NodeJS Dependencies'
```

List files in a directory

```
- bash: ls
      workingDirectory: '$(Build.SourcesDirectory)/$(myDirectory)'
      displayName: 'List myDirectory'
``` 


## Artifacts

When a job is completed, all the actions in its steps are cleaned up. If a user builds something with a Task or Script step in Job 1 (e.g. node_modules via npm i). Those built files will be removed at Job 1 completion and will be absent at the start of Job 2. In order to pass files from Job 1 to Job 2, a user must build and publish artifact containing those files at the end of Job 1. Afterwards, that artifact must be downloaded at the beginning of Job 2.

e.g. 

Job 1

```
- task: CopyFiles@2
  displayName: 'Save hello.txt as artifact'
  inputs:
    contents: '$(myDirectory)/hello.txt'
    targetFolder: '$(Build.ArtifactStagingDirectory)'

- publish: '$(Build.ArtifactStagingDirectory)/$(myDirectory)'
  displayName: 'Publish artifact'
  artifact: 'hello'
```

Job 2

```
- task: DownloadPipelineArtifact@2
      inputs:
        artifact: 'hello'
        path: '$(Build.SourcesDirectory)/$(myDirectory)'
```

## Pipeline Caching

If a pipeline constantly builds the same files (e.g. the same node_modules via npm i), the resulting built files can be cached with a hash. The next run, those files can be reused to gain time.

e.g. (NodeJS-specific)
```
- task: Cache@2
      inputs:
        key: 'version1 | "$(Agent.OS)" | $(myDirectory)/package.json'
        restoreKeys: |
          npm | "$(Agent.OS)"
        path: '$(Build.SourcesDirectory)/$(myDirectory)/node_modules'
        cacheHitVar: CACHE_RESTORED
```