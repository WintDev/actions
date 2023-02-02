# actions
This repository contains reusable workflows and common actions used by the **WintDev** organization CI/CD.

## Workflow versions
There are four versions available.
- v1
  - Use this version for managing Azure Function Apps targeting Azure Functions v3 with code targeting .net versions < net6.
- v2 (**depreciated**)
  - Use this version for managing Azure Function Apps targeting Azure Functions v4 with code targeting .net6. This version is kept for legacy 
- v3
  - Use this version for managing Azure Function Apps targeting Azure Functions v4 with code targeting .net6.
- v4
  - Use this version for managing Azure Function Apps targeting Azure Functions v4 with code targeting .net6. __and__ when working with [environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment#using-an-environment) when deploying web- or function apps.

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
- app-name
  - The name of the azure functions app.
        required: true
        type: string
- environment
  - The name of the environment where the function app would be deployed.
    **Note:** This input is available from v4 onwards.
- package-name
  - This is the location in your project to be published. By default, this value is set to 'app'.

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
      environment: Production      
      package-name: wint-myservice-package

    secrets:
      publish-profile: ${{ secrets.MYSERVICE_PRODUCTION_ENVIRONMENT_PUBLISH_PROFILE }} # The publish profile should be downloaded from the azure portal and stored as an environment secret in the environment matching the 'environment' input, i.e. 'Production' in this example.
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
- environment:
  - The name of the environment where the web app would be deployed.
    **Note:** This input is available from v4 onwards.
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
      environment: Production
      package-name: wint-mywebapi-package
      slot-name: staging # Note: Deploy is made to the staging slot for verification

    secrets:
      publish-profile: ${{ secrets.MYWEBAPI_PRODUCTION_ENVIRONMENT_PUBLISH_PROFILE }} # The publish profile should be downloaded from the azure portal and stored as an environment secret in the environment matching the 'environment' input, i.e. 'Production' in this example.
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

### <a name="testappwindows"></a>test-app-windows (test-app-windows.yml)
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

### <a name="build-push-container"></a>build-push-container (build-push-container.yml)
Call this workflow to build and push a Docker Container to an Azure Container Registry (ACR).

**NOTE:** By default this workflow will push the built contaier to the [WintContainer](https://portal.azure.com/#@wint.se/resource/subscriptions/274a26c3-e153-4564-b062-15c5e39cdfdb/resourceGroups/WintContainer/providers/Microsoft.ContainerRegistry/registries/WintContainer/overview) ACR. 

#### Inputs
- login-server
  -The **URL** of the ACR. It will default to [wintcontainer](wintcontainer.azurecr.io).
- dockerfile
  - The path to the Dockerfile that contains the instruction for the build. The default value is **Dockerfile**, indicating that the file is located in the workig directory.
- working-directory
  - The directory from where the **docker build** commad would be issued. The default value is ., indicating that the **docker build** command would be issued from the repository root.
- container-name
  - The name that together with the calculated tag value will make upp the --tag argument in the container build. If not specified, the repository name will be used as the container name.
- tag
  - A hard-coded tag value. If not specified, the container tag will be calculated by the ref of build (i.e. the branch name).
- environment
  - An environment value used to populate the **ASPNETCORE_ENVIRONMENT** evironment variable in the built container. The default value is **Production**.
  **Note:** The __environment__ input is supplied to the **docker build** command as a **--build-arg**. The target **Dockerfile** has to handle the argument for it to have any effect. The sample [Dockerfile](https://github.com/WintDev/Wint.Infrastructure/blob/main/Samples/EF/Wint.Infrastructure.Samples.WebApiSample/Dockerfile) for the EF WebApi is an example of such handling.

**Note:** A common scenario is that Visual Studio has generated the Dockerfile and thus placed it in the folder of the assembly where the apps startup code is located. This has the benefit of being able to use some of the Dockerfile scaffolding when debugging the app from within a container (i.e. debugging with Docker support). The downside is that the workflow has to emulate the Visual Studio build process to match. In this scenario, the **working-directory** input would be specified to the directory of the Visual Studio solution file and the **Dockerfile** input would be specified to the apps corresponding Dockerfile.

The [workflow file](https://github.com/WintDev/Wint.Infrastructure/blob/main/.github/workflows/build-EF-WebApiSample.yml) to build the sample EF WebApi container uses a **Dockerfile** originally created by Visual Studio and as such is an example of the scenario described above.

Below is an example of a workflow file where the Dockerfile is located at the repository root instead.
```yaml
jobs:
  build-push:
    name: Build and Push app container
    uses: WintDev/actions/.github/workflows/build-push-container.yml@v4
    with:
      login-server: wintcontainer.azurecr.io
      dockerfile: Dockerfile
      container-name: my.container
```

#### Secrets
- username
  - The username used to access the ACR. When the target ACR is the WintContainer, the organization secret **WINTCONTAINER_USERNAME** should be used.
- password
  - The password used to access the ACR. When the target ACR is the WintContainer, the organization secret **WINTCONTAINER_PASSWORD** should be used.
- nuget-read-pat
  - The Personal Access Token used to access the WintDev internal nuget feed during restore.

### <a name="build-deploy-container"></a>build-deploy-container (build-deploy-container.yml)
Call this workflow to build a Docker Container, push to an Azure Container Registry (ACR) and deploy a Container App.

#### Inputs
- login-server
  -The **URL** of the ACR. It will default to [wintcontainer](wintcontainer.azurecr.io).
- dockerfile
  - The path to the Dockerfile that contains the instruction for the build. The default value is **Dockerfile**, indicating that the file is located in the workig directory.
- working-directory
  - The directory from where the **docker build** commad would be issued. The default value is ., indicating that the **docker build** command would be issued from the repository root.
- container-name
  - The name that together with the calculated tag value will make upp the --tag argument in the container build. If not specified, the repository name will be used as the container name.
- tag
  - A hard-coded tag value. If not specified, the container tag will be calculated by the ref of build (i.e. the branch name).
- environment
  - An environment value used to populate the **ASPNETCORE_ENVIRONMENT** evironment variable in the built container. The default value is **Production**.
  **Note:** The __environment__ input is supplied to the **docker build** command as a **--build-arg**. The target **Dockerfile** has to handle the argument for it to have any effect. The sample [Dockerfile](https://github.com/WintDev/Wint.Infrastructure/blob/main/Samples/EF/Wint.Infrastructure.Samples.WebApiSample/Dockerfile) for the EF WebApi is an example of such handling.

**Note:** This workflow uses the [Azure CLI az containerapp up](https://learn.microsoft.com/en-us/cli/azure/containerapp?view=azure-cli-latest#az-containerapp-up) command. The inputs below are passed to this action.

- container-app-name:
  - The name of the Container App that will be created or updated
- container-app-resource-group:
  - The resource group that the Container App will be created in, or currently exists in.
- container-app-environment:
  - The name of the Container App environment to use with the application. The default value is WintContainerEnvironment.
- container-app-location:
  - The location that the Container App (and other created resources) will be deployed to. The default value is **North Europe**.
- container-app-env-vars:
  - A list of environment variables for the container. Space-separated values in 'key=value' format. Use an empty string to clear existing values. The prefix value 'secretref:' is used to reference a secret.
  **Example:** MY_ENV_KEY_0=ENV_VALUE_0 MY_ENV_KEY_1=ENV_VALUE_1 MY_ENV_KEY_2=secretref:MY_ENV_SECRET_2

#### Secrets
- username
  - The username used to access the ACR. When the target ACR is the __WintContainer__, the organization secret **WINTCONTAINER_USERNAME** should be used.
- password
  - The password used to access the ACR. When the target ACR is the __WintContainer__, the organization secret **WINTCONTAINER_PASSWORD** should be used.
- nuget-read-pat
  - The Personal Access Token used to access the WintDev internal nuget feed during restore.

**Note:** See the note for dockerfile location scenario in [build-push-container](#a-namebuild-push-containerabuild-push-container-build-push-containeryml) action for information on dockerfile location alternatives.