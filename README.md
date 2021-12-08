# actions
This repository contains reusable workflows and common actions used by the **WintDev** organization CI/CD.

## Workflows
### build.app.yml
Call this workflow to build, test and publish the build artifacts to a downloadable artifact.

The artifacts produced has a retention of one day.

#### Inputs
- solution-name
  - The name of solution to build, test and publish. Example **mySolution.sln**.  
- artifacts-prefix
  - The prefix of the artifacts that will contain the function app bits. 
- package-name
  - The name of the package contained in the artifacts published.
#### Secrets
- nuget-read-pat
  - The Personal Access Token used to access the WintDev internal nuget feed.
