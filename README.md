# Nexus-Azure-Devops


write a azure devops pipeline to restore build pack publish and publish artifact. 
the pipeline will have 3 branches main, develop and release.
versioning should happen when packet is being packed.
if the commit if from develop branch than versoing format should be,
<Project-name>-<build-id>-<branch-name>. if commit is from develop branch than create a folder named demo only once and then whenever pipeline gets triggered for develop branch new packets versions should be stored under that demo folder
This is expected output helloworld-<build-id>-develop.nupkg.
if the commit if from main branch than versoing format should be,
<Project-name>-<tag>-<version>. if commit is from main branch than create a new folder for every new version to store packets.
this is my basic skeletion code which is able to push code


the logic for naming nupkg is incorrect when packed by develop branch. expected is <projevct-name>-<build-id>-<branch-name>.nupkg. here are expected example output for reference helloworld-324-develop.nupkg helloworld-321-develop.nupkg. in this format it should be packed. fix the logic

trigger:
  branches:
    include:
      - main  

pool:
  vmImage: 'windows-latest'  

variables:
  RestoreBuildProjects: '**/*.csproj'  
  BuildConfiguration: 'Release'  
  NuGetPackageOutput: '$(Build.ArtifactStagingDirectory)/nuget'  

steps:
  - script: dir /s
    displayName: 'List all files in the repository'

  - task: UseDotNet@2
    inputs:
      packageType: 'sdk'
      version: '8.x'  
      installationPath: $(Agent.ToolsDirectory)/dotnet

  - task: DotNetCoreCLI@2
    displayName: Restore
    inputs:
      command: restore
      projects: '$(RestoreBuildProjects)'  

  - task: DotNetCoreCLI@2
    displayName: Build
    inputs:
      projects: '$(RestoreBuildProjects)'
      arguments: '--configuration $(BuildConfiguration)'

  - task: DotNetCoreCLI@2
    displayName: Pack
    inputs:
      command: pack
      packagesToPack: '$(RestoreBuildProjects)'  
      versioningScheme: 'off'  
      outputDir: '$(NuGetPackageOutput)'  

  - task: DotNetCoreCLI@2
    displayName: Test
    inputs:
      command: test
      projects: '$(TestProjects)'  
      arguments: '--configuration $(BuildConfiguration)'

  - task: DotNetCoreCLI@2
    displayName: Publish
    inputs:
      command: publish
      publishWebProjects: True
      arguments: '--configuration $(BuildConfiguration) --output $(Build.ArtifactStagingDirectory)'
      zipAfterPublish: True

  - task: PublishBuildArtifacts@1
    displayName: 'Publish Artifact'
    inputs:
      PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    condition: succeededOrFailed()


  - task: NuGetCommand@2
    inputs:
      command: 'push'
      packagesToPush: '$(Build.ArtifactStagingDirectory)/**/*.nupkg;!$(Build.ArtifactStagingDirectory)/**/*.symbols.nupkg'
      nuGetFeedType: 'external'
      publishFeedCredentials: 'develop-con'
note the code should be modified by you according to the requirments specified above.
