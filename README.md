# actions
This repository contains reusable workflows and common actions used by the **WintDev** organization CI/CD.

## Workflows
### build-test-upload (build.app.yml)
Call this workflow to build, test and publish the build artifacts to a downloadable artifact.

The artifacts produced has a retention of one day.

#### Inputs
- solution-name
  - The name of solution to build, test and publish. Example **mySolution.sln**.
- assembly-url
  - The path to the entry assembly of the app. This is the input to the publish command that will produce the artifacts of the build. 
- artifacts-prefix
  - The prefix of the artifacts that will contain the function app bits. 
- package-name
  - The name of the package contained in the artifacts published.
#### Secrets
- nuget-read-pat
  - The Personal Access Token used to access the WintDev internal nuget feed during restore.

#### Example
The example below builds a solution, MyService.sln and publishes an artifact for the app called MyService
```yaml
jobs:
  build_app:
    name: Build (MySolution)
    uses: WintDev/actions/.github/workflows/build-app.yml@v1
    with:
      solution-name: Wint.MyService.sln
      assembly-url: Wint.MyService/Wint.MyService.csproj
      artifacts-prefix: wint-myservice
      package-name: wint-myservice-package
    secrets:
      nuget-read-pat: ${{ secrets.NUGET_READ_PAT }}
```

### build-pack-upload (build-package.yml)
Call this workflow to push a nuget package from a packable assembly in the repository.
#### Inputs
- assembly-url:
  - The path to the assembly that would be packed. N.B: This dictates the name of the nuget package that will be pushed to the internal repository.
- assembly-prefix
  - The prefix for the build artifacts to sign.
- artifacts-prefix
  - The name of the artifacts that contain the .nupkg files produced by the workflow.'

#### Secrets
- codesigning-cert
  - The base64 string that describe the code signing certificate.
- codesigning-cert-pwd:
  - The password to the private key of the code signing certificate.
- nuget-read-pat
  - The Personal Access Token used to access the WintDev internal nuget feed during restore. 

#### Example
The example below builds a solution, MyService.sln and publishes an artifact for the app called MyService
```yaml
jobs:
  build_packages:
    name: Build package assembly (Wint.MyService.Model)
    uses: WintDev/actions/.github/workflows/build-packages.yml@v1
    with:
      assembly-url: Wint.MyService.Models/Wint.MyService.Models.csproj
      assembly-prefix: 'Wint.MyService'
      artifacts-prefix: wint-myservice-package
    secrets:
      codesigning-cert: ${{ secrets.CODESIGNING_WINT_SE }}    
      codesigning-cert-pwd: ${{ secrets.CODESIGNING_WINT_SE_PWD }}
      nuget-read-pat: ${{ secrets.NUGET_READ_PAT }}
```
