# actions
This repository contains reusable workflows and common actions used by the **WintDev** organization CI/CD.

## Workflow versions
There are three versions available.
- v1
  - Use this version for managing Azure Function Apps targeting Azure Functions v3 with code targeting .net versions < net6.
- v2 (**depreciated**)
  - Use this version for managing Azure Function Apps targeting Azure Functions v4 with code targeting .net6. This version is kept for legacy 
- v3
  - Use this version for managing Azure Function Apps targeting Azure Functions v4 with code targeting .net6.

**Note:** v2 is kept for legacy support. It _may_ be used when there is a need to manage pushing nuget packages and deployment of Azure Function apps manually in the workflow.

## Workflows
**Note:** Because of the [github actions reusable workflow limitations](https://docs.github.com/en/actions/learn-github-actions/reusing-workflows#limitations), there are decicated shared workflows for testing on the Windows and Ubuntu environments respectively. For more information on this,  see: [test-app-windows](#testappwindows), and [test-app-ubuntu](#testappubuntu).

### build-test-upload (build.test.upload.app.yml)
Call this workflow to build, test and publish the built bits to a downloadable artifact.

The artifacts produced has a retention of one (1) day.

#### Inputs
- solution-name
  - The name of solution to build, test and publish. Example **mySolution.sln**.
- assembly-url
  - The path to the entry assembly of the app. This is the input to the publish command that will produce the artifacts of the build. 
- artifacts-name
  - The name of the artifacts that will contain the function app bits. 
- package-name
  - The name of the package contained in the artifacts published.
#### Secrets
- nuget-read-pat
  - The Personal Access Token used to access the WintDev internal nuget feed during restore.

#### Example
The example below builds a solution, MyService.sln and publishes an artifact for the app called MyService
```yaml
jobs:
  build:
    name: Build (MySolution)
    uses: WintDev/actions/.github/workflows/build.test.upload.app.yml@v3
    with:
      solution-name: Wint.MyService.sln
      assembly-url: Wint.MyService/Wint.MyService.csproj
      artifacts-name: 'wint-myservice-${{ github.run_id }}' # Note: We use the run id to make the artifacts from the build unique
      package-name: wint-myservice-package
    secrets:
      nuget-read-pat: ${{ secrets.NUGET_READ_PAT }}
```

### deploy-function-app (deploy-function-app.yml)
Call this workflow to publish built artifacts to an azure function app.

#### Inputs
- artifacts-name:
  - The name of the artifacts that contain the app bits to deploy.
- assembly-prefix:
  - The prefix for the build artifacts to sign, e.g. **Wint.MyDomain**
#### Secrets
- publish-profile
  - The publish profile of the azure function app.
- codesigning-cert
  - The certificate to use for code signing in base64.
- codesigning_pwd:
  - The password to the private key of the signing certificate.

#### Example
The example below builds a solution, MyService.sln and uses the build artifacts to deploy an Azure Function App **MyService**
```yaml
jobs:
  build:
    name: Build (MySolution)
    uses: WintDev/actions/.github/workflows/build.test.upload.app.yml@v3
    with:
      solution-name: Wint.MyService.sln
      assembly-url: Wint.MyService/Wint.MyService.csproj
      artifacts-name: 'wint-myservice-${{ github.run_id }}' # Note: We use the run id to make the artifacts from the build unique
      package-name: wint-myservice-package
    secrets:
      nuget-read-pat: ${{ secrets.NUGET_READ_PAT }}

  deploy:
    name: Deploy (MyApp)
    needs: build
    uses: WintDev/actions/.github/workflows/deploy-function-app.yml@v3
    with:
      artifacts-name: 'wint-myservice-${{ github.run_id }}'
      assembly-prefix: 'Wint.MyService'
      app-name: MyService
      package-name: wint-myservice-package

    secrets:
      publish-profile: ${{ secrets.MYSERVICE_PUBLISH_PROFILE }}
      codesigning-cert: ${{ secrets.CODESIGNING_WINT_SE }}
      codesigning_pwd: ${{ secrets.CODESIGNING_WINT_SE_PWD }}
```

### deploy-web-app (deploy-web-app.yml)
Call this workflow to publish built artifacts to an azure web app (asp .net core web app).

#### Inputs
- artifacts-name:
  - The name of the artifacts that contain the app bits to deploy.
- assembly-prefix:
  - The prefix for the build artifacts to sign, e.g. **Wint.MyDomain**
- app-name:
  - The name of the azure web app
- package-name
  - The location in your project to be published.
- slot-name
  - The name of the target slot where the app would be published.

#### Secrets
- publish-profile
  - The publish profile of the azure web app.
- codesigning-cert
  - The certificate to use for code signing in base64.
- codesigning_pwd:
  - The password to the private key of the signing certificate.

#### Example
The example below builds a solution, MyWebApi.sln and uses the build artifacts to deploy an Azure Web App **MyWebApi** to a slot named __staging__.
```yaml
jobs:
  build:
    name: Build (MyWebApi)
    uses: WintDev/actions/.github/workflows/build.test.upload.app.yml@v3
    with:
      solution-name: Wint.MyWebApi.sln
      assembly-url: Wint.MyWebApi/Wint.MyWebApi.csproj
      artifacts-name: 'wint-mywebapi-${{ github.run_id }}' # Note: We use the run id to make the artifacts from the build unique
      package-name: wint-mywebapi-package
    secrets:
      nuget-read-pat: ${{ secrets.NUGET_READ_PAT }}

  deploy:
    name: Deploy (MyWebApi)
    needs: build
    uses: WintDev/actions/.github/workflows/deploy-web-app.yml@v3
    with:
      artifacts-name: 'wint-mywebapi-${{ github.run_id }}'
      assembly-prefix: 'Wint.MyWebApi'
      app-name: MyWebApi
      package-name: wint-mywebapi-package
      slot-name: staging # Note: Deploy is made to the staging slot for verification

    secrets:
      publish-profile: ${{ secrets.MYWEBAPI_PUBLISH_PROFILE }} # The publish profile should be downloaded from the azure portal and stored as a repository secret.
      codesigning-cert: ${{ secrets.CODESIGNING_WINT_SE }}
      codesigning_pwd: ${{ secrets.CODESIGNING_WINT_SE_PWD }}
```

### build-pack-push-package (build-pack-push-package.yml)
Call this workflow to push a nuget package from a packable assembly in the repository.
#### Inputs
- assembly-url:
  - The path to the assembly that would be packed. **N.B:** This dictates the name of the nuget package that will be pushed to the internal repository.
- assembly-prefix
  - The prefix for the build artifacts to sign.

#### Secrets
- codesigning-cert
  - The base64 string that describe the code signing certificate.
- codesigning-cert-pwd:
  - The password to the private key of the code signing certificate.
- nuget-read-pat
  - The Personal Access Token used to access the WintDev internal nuget feed during restore. 

#### Example
The example below pushes a new package version of the nuget package **Wint.MyService.Models**.
```yaml
jobs:
  build_packages:
    name: Build package assembly (Wint.MyService.Model)
    uses: WintDev/actions/.github/workflows/build-pack-push-package.yml@v3
    with:
      assembly-url: Wint.MyService.Models/Wint.MyService.Models.csproj
      assembly-prefix: 'Wint.MyService'
    secrets:
      codesigning-cert: ${{ secrets.CODESIGNING_WINT_SE }}    
      codesigning-cert-pwd: ${{ secrets.CODESIGNING_WINT_SE_PWD }}
      nuget-read-pat: ${{ secrets.NUGET_READ_PAT }}
```

### <a name="testappubuntu"></a>test-app-ubuntu (test-app-ubuntu.yml) (#test-app-ubuntu)
Call this workflow to build and test a solution on the Ubuntu operating system.

#### Inputs
- solution-name
  - The name of solution to build, test and publish. Example **mySolution.sln**.

#### Secrets
- nuget-read-pat
  - The Personal Access Token used to access the WintDev internal nuget feed during restore. 

#### Example
The example below builds a solution, MyService.sln and publishes an artifact for the app called MyService
```yaml
jobs:
  build:
    name: test-ubuntu
    uses: WintDev/actions/.github/workflows/test-app-ubuntu.yml@v3
    with:
      solution-name: Wint.MyService.sln
    secrets:
      nuget-read-pat: ${{ secrets.NUGET_READ_PAT }}
```

### <a name="testappwindows"></a>test-app-windows (test-app.windows.yml)
Call this workflow to build and test a solution on the Windows operating system.

#### Inputs
- solution-name
  - The name of solution to build, test and publish. Example **mySolution.sln**.

#### Secrets
- nuget-read-pat
  - The Personal Access Token used to access the WintDev internal nuget feed during restore. 

#### Example
The example below builds a solution, MyService.sln and publishes an artifact for the app called MyService
```yaml
jobs:
  build:
    name: test-windows
    uses: WintDev/actions/.github/workflows/test-app-windows.yml@v3
    with:
      solution-name: Wint.MyService.sln
    secrets:
      nuget-read-pat: ${{ secrets.NUGET_READ_PAT }}
```
