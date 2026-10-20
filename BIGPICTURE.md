### Mermaid Legend:

```mermaid
%% ==== OVERVIEW GRAPH ====
flowchart LR


Domain["Logical Domain"]:::DomainStyle
Entity["Single Entity"]:::EntitySyle

Domain -- physical link --> Entity
Domain -. logical link .-> Entity

classDef DomainStyle rx:12,ry:12,fill:none,stroke-dasharray:8
classDef EntitySyle rx:12,ry:12,fill:none
```

# CrownLabs Big Picture
The following will be placed in **home**

```mermaid
flowchart LR

User@{shape: trap-b, label: "User Browser⁴"}

subgraph Cluster["Kubernetes Cluster"]
    %% Cells
    Argo["Argo/Helm¹"]
    Release["CL Argo Application¹"]

    %% subgraphs
    subgraph ReleaseNS["CL Release Namespace¹"]
        Controllers@{shape: docs, label: "Controller Set¹"}
        Frontend["Frontend-app⁴"]:::FrontendStyle
        GraphQL["GraphQL Relay⁵"]
        Bastion["SSH Bastion"]
    end
    WorkspaceCR["`Workspace CR
        [cluster-wide]
        _workspace: abc_`"]:::CustomResourceStyle
    Controllers -- manages --> WorkspaceCR
    subgraph WorkspaceNS["`Workspace Namespace:
    _workspace-abc_ ³`"]
        direction LR
        TemplateCR["`Template CR
            template: _foo_`"]:::CustomResourceStyle
    end
    TenantCR["`Tenant CR
        [cluster-wide]
        _tenant: xyz-efg_`"]:::CustomResourceStyle
    subgraph TenantNS["`Tenant Namespace: _tenant-xyz-efg_ ²`"]
        InstanceCR["`Instance CR
            _instance: bar_`"]:::CustomResourceStyle
        InstanceEnv@{shape: docs, label: "bar Environments"}

    end
    subgraph K8S["K8S Provided Infratructure"]
        APIServer["API Server"]
        GWAPI["Gateway Balancer"]
    end
end

Controllers -- manages --> TenantCR
Argo .-> Release
Release -. becomes .-> ReleaseNS
Frontend -. exposed through .-> GWAPI
Frontend -. delivered to .-> User
GraphQL .-> APIServer
GWAPI ---> InstanceEnv
WorkspaceCR -- reconciles to --> WorkspaceNS
TenantCR -- reconciles to --> TenantNS
User --> GWAPI
User -. interacts with .-> GraphQL
User -. interacts with .-> InstanceEnv
InstanceCR -. references to ..-> TemplateCR
InstanceCR --> InstanceEnv
GraphQL -. exposed through .-> GWAPI
User --> Bastion
Bastion  --> InstanceEnv

classDef ControllerStyle stroke:#FF8000,rx:12,ry:12,fill:none
classDef WorkspaceStyle stroke:#0A6522,rx:12,ry:12,fill:none
classDef CustomResourceStyle stroke:#FF0000,rx:12,ry:12,fill:none
classDef TenantStyle stroke:#2222FF,rx:12,ry:12,fill:none
classDef EnvironmentStyle stroke:#A865B5,rx:12,ry:12,fill:none
classDef FrontendStyle stroke:#FDDCD7,rx:12,ry:12,fill:none
classDef ClusterStyle stroke:#80FF00,rx:12,ry:12,fill:none

classDef GenericDomainStyle fill:none,rx:12,ry:12,stroke-dasharray:8
classDef WorkspaceNSStyle stroke:#0A6522,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef TenantNSStyle stroke:#2222FF,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef ReleaseNSStyle stroke:#FF8000,rx:12,ry:12,fill:none,stroke-dasharray:8

class K8S GenericDomainStyle
class ReleaseNS ReleaseNSStyle
class Controllers ControllerStyle
class WorkspaceNS WorkspaceNSStyle
class TemplateEnv,InstanceEnv EnvironmentStyle
class TenantNS TenantNSStyle
class Cluster ClusterStyle
```
¹More about Deployment+Gateway [here](BIGPICTURE.md#crownlabs-deployment).  
²More about Tenant [here](BIGPICTURE.md#tenant-business-logic).  
³More about Workspace [here](BIGPICTURE.md#workspace-business-logic).  
⁴More about Frontend [here](BIGPICTURE.md#frontend-logic)  
⁵More about QLKube  


## CrownLabs Deployment Releases
The following will be placed in **home/deploy/crownlabs**

```mermaid
flowchart TB

ArgoHelm["Helm / ArgoCD"] 
ProdDeploy["`_crownlabs-production_
    Production`"]
PreProdDeploy["`_crownlabs-pre-production_
    Pre-production`"]
StagingDeploy["`_crownlabs-staging-123_
    Staging (PR-associated)`"]
subgraph Controllers["CL Release Namespace"]
    direction TB
    subgraph Operator["`CL Operator
    (_operator-selector = prod_)`"]
        TenantController["Tenant Controller"]
        WorkspaceController["Workspace Controller"]
        ShVolController["Shared Volume Controller"]
        PMPController["PMP Controller"]
    end
    subgraph InstOp["`Instance Operator
    (_operator-selector = prod_)`"]
        InstanceController["Instance Controller"]
        InstanceSnapshotController["Instance Snapshot Controller"]
    end
    subgraph InstAutOp["`Instance Automation Controller
    (_operator-selector = prod_)`"]
        ExpirationController["Expiration Controller"]
        InactivityController["Inactivity Controller"]
        SubmissionController["Submission Controller"]
        TerminationController["Termination Controller"]
    end
    subgraph BastionOp["Bation Operator"]
        SSHController["SSH Bastion"]
        BastionController["Bastion Controller"]
    end
    subgraph Others[" "]
        FrontendController["CrownLabs Frontend²"]
        WebSSHController["WebSSH"]
        QLKubeController["QLKube"]
    end
    subgraph Others2[" "]
        RBAC@{shape: docs, label: "RBAC 
        _when enabled_
        Shared Cluster Roles"}
        Gateway@{shape: docs, label: "Gateway/Exposition Related Resources¹"}
    end
end

ArgoHelm --> ProdDeploy
ArgoHelm --> PreProdDeploy
ArgoHelm --> StagingDeploy
ProdDeploy -- becomes --> Controllers

classDef ReleasesNSStyle stroke:#FF8000,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef OperatorStyle stroke:#38677B,fill:none
classDef InvisibleStyle stroke:none,fill:none

class Controllers ReleasesNSStyle
class Others,Others2 InvisibleStyle
class Operator,InstOp,InstAutOp,BastionOp,FrontendController,WebSSHController,QLKubeController OperatorStyle
```
¹Gateway
²Frontend

### Physical Flow

```mermaid
flowchart LR

User@{shape: trap-b, label: "User Browser"}

subgraph ReleaseNS["CL Release Namespace"]
    FrontendApp["Frontend-app"]:::FrontendStyle
end
subgraph EnvoyNS["`envoy-gateway-system Namespace`"]
    LoadBalancer["LoadBalancer"]
    Deployment["Deployment"]
    Proxy["Proxy"]
end
subgraph TenantNS["`Tenant Namespace: _tenant-xyz_`"]
    TargetService["Target Service"]
end

User --> FrontendApp
User --> LoadBalancer
FrontendApp --> LoadBalancer
LoadBalancer --> Proxy
Proxy --> TargetService

Deployment -. creates/updates .-> LoadBalancer
Deployment -. creates/updates .-> Proxy

classDef ReleaseNSStyle stroke:#FF8000,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef EnvoyNSStyle stroke:#38677B,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef TenantNSStyle stroke:#2222FF,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef TenantStyle stroke:#2222FF,rx:12,ry:12,fill:none
classDef FrontendStyle stroke:#FDDCD7,rx:12,ry:12,fill:none

class ReleaseNS ReleaseNSStyle
class EnvoyNS EnvoyNSStyle
class TenantNS TenantNSStyle
class TargetService TenantStyle
```

### Logical Flow

```mermaid
flowchart LR

User@{shape: trap-b, label: "User Browser"}

subgraph ReleaseNS["CL Release Namespace"]

    Gateway["`Gateway
        (Envoy Gateway API)`"]
    FrontendApp["Frontend-app"]:::FrontendStyle

    subgraph SecPols["Security Policies"]
        SecPolG["Security Policy"]
        SecPolDummy["Dummy Security Policy"]
    end

    subgraph Routes["HTTPRoutes"]
        CallbackRoute["`callback HTTPRoute
            _/app/instauth/callback_`"]
        subgraph PublicRoutes["Public Routes"]
            FrontendRoute["`Frontend-app HTTPRoute
                _/_`"]
            WebSSHRoute["`WebSSH HTTPRoute
                _/webssh_`"]
            QLKubeRoute["`QLKube HTTPRoute
                _/graph_`"]
        end
    end
end
subgraph EnvoyNS["`envoy-gateway-system Namespace`"]
    Deployment["Deployment"]
end
subgraph TenantNS["`Tenant Namespace: _tenant-xyz_`"]
    OtherRoute["`another HTTPRoute
        _/instance/{uuid}/{env-name}_`"]
    TargetService["Target Service"]
end

User -.-> FrontendApp
User -.-> Gateway
FrontendApp -.-> Gateway
Gateway -.-> Routes
Gateway -.-> OtherRoute
Gateway -. authenticated through .-> SecPols
SecPolG -.-> CallbackRoute
SecPolG -.-> OtherRoute
SecPolDummy -.-> PublicRoutes

Deployment -. creates/updates .-> Gateway
Deployment -. creates/updates .-> Routes
Deployment -. creates/updates .-> SecPols
Routes -.-> TargetService
OtherRoute -.-> TargetService

classDef ReleaseNSStyle stroke:#FF8000,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef EnvoyNSStyle stroke:#38677B,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef TenantNSStyle stroke:#2222FF,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef GenericDomainStyle fill:none,rx:12,ry:12,stroke-dasharray:8
classDef TenantStyle stroke:#2222FF,rx:12,ry:12,fill:none
classDef FrontendStyle stroke:#FDDCD7,rx:12,ry:12,fill:none

class ReleaseNS ReleaseNSStyle
class EnvoyNS EnvoyNSStyle
class TenantNS TenantNSStyle
class Routes,PublicRoutes,SecPols GenericDomainStyle
class OtherRoute,TargetService TenantStyle
```

## Frontend Logic
The following will be placed in **home/frontend-app**

```mermaid
flowchart LR

Home["`Home Page
    _crownlabs.polito.it_`"]:::FrontendStyle
Login["Login Page"]:::FrontendStyle
Frontend["`Front-end
    _crownlabs.polito.it/app_`"]:::FrontendStyle
Remote["Remote Desktop"]
OIDC["`OIDC Server
    (Authentication)`"]
Graph["`GraphQL
    _crownlabs.polito.it/graph_`"]:::FrontendStyle
GWAPI["Gateway (Balancer)"]

Home --> Login
Login --> Frontend
Login --> OIDC
Frontend --> Graph
Frontend -. connects .-> Remote
Frontend -. connects .-> GWAPI
Remote --> GWAPI

classDef FrontendStyle stroke:#FDDCD7,rx:12,ry:12,fill:none
```

## Workspace Business Logic
The following will be placed in **home/operators/operator/workspace**

```mermaid
flowchart LR

Keyclock
WorkspaceController["Workspace Controller"]:::ControllerStyle
WorkspaceCR["`Workspace CR
    _workspace: abc_`"]:::CustomResourceStyle
subgraph WorkspaceNS["`Workspace Namespace: _workspace-abc_`"]
    subgraph TemplateFoo["`Template: _foo_`"]
        EnvContainer["`Environment 1
            _(container-based)_`"]:::EnvironmentStyle
        EnvVM["`Environment 2
            _(vm-based)_`"]:::EnvironmentStyle
    end

    TemplateOther["`Another Template
        _(does nothing if not referenced in an instance)_`"]
end

WorkspaceController -- reconciles --> WorkspaceCR
WorkspaceCR --> WorkspaceNS
WorkspaceController -- groups provisioning --> Keyclock

classDef ControllerStyle stroke:#FF8000,rx:12,ry:12,fill:none
classDef TemplateStyle stroke:#0A6522,rx:12,ry:12,fill:none
classDef CustomResourceStyle stroke:#FF0000,rx:12,ry:12,fill:none
classDef EnvironmentStyle stroke:#A865B5,rx:12,ry:12,fill:none
classDef WorkspaceNSStyle stroke:#0A6522,rx:12,ry:12,fill:none,stroke-dasharray:8

class WorkspaceNS WorkspaceNSStyle
class TemplateFoo,TemplateOther TemplateStyle
```


## Tenant Business Logic
The following will be placed in **home/operators/operator/tenant**

```mermaid
flowchart LR

TenantController["Tenant Controller"]:::ControllerStyle
BastionController["Bastion Controller"]:::ControllerStyle
SharedVolumeController["Shared Volume Controller"]:::ControllerStyle

TenantCR["`Tenant CR
_tenant: xyz.efg_`"]:::CustomResourceStyle
subgraph TenantNS["`Tenant Namespace: _tenant-xyz-efg_ (has label for _operator-selector=production_)`"]
    InstanceCR["`Instance CR
        _instance: bar_`"]:::CustomResourceStyle
    ShVol["Shared Volume"]
    ContainerEnv["Container-based Environment"]:::EnvironmentStyle
    VMEnv["VM-based Environment"]:::EnvironmentStyle
end

subgraph TemplateFoo["`Template: _foo_`"]
    EnvContainer["`Environment 1
        _(container-based)_`"]:::EnvironmentStyle
    EnvVM["`Environment 2
        _(vm-based)_`"]:::EnvironmentStyle
end

TenantController -- reconciles --> TenantCR
BastionController -. watches .-> TenantCR
TenantCR -- manages --> TenantNS
SharedVolumeController -- reconciles --> ShVol
InstanceCR -. references .-> TemplateFoo
EnvContainer -. generates .-> ContainerEnv
EnvVM -. generates .-> VMEnv
ContainerEnv -. attaches .-> ShVol
VMEnv -. attaches .-> ShVol

classDef ControllerStyle stroke:#FF8000,rx:12,ry:12,fill:none
classDef CustomResourceStyle stroke:#FF0000,rx:12,ry:12,fill:none
classDef EnvironmentStyle stroke:#A865B5,rx:12,ry:12,fill:none
classDef WorkspaceNSStyle stroke:#0A6522,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef TenantNSStyle stroke:#2222FF,rx:12,ry:12,fill:none,stroke-dasharray:8

class TenantNS TenantNSStyle
class TemplateFoo WorkspaceNSStyle
```

## Instances Deep Dive
The following will be placed in **home/operators/.../instctl**

```mermaid
flowchart LR

InstanceController["Instance Controller"]:::ControllerStyle
InstanceAutomationController["Instance Automation Controller"]:::ControllerStyle
InstanceSnapshotController["Instance Snapshot Controller"]:::ControllerStyle

subgraph TenantNS["`Tenant Namespace: _tenant-xyz-efg_`"]
    InstanceCR["`Instance CR
        _instance: bar_`"]:::CustomResourceStyle
    subgraph Env1ContainerInst["`Instanced Environment 1 _(container-based)_`"]
        ExpositionEnv1["Ingress / HTTPRoute"]
        ServiceEnv1["Service"]
        DeploymentEnv1["Deployment"]

        DeploymentEnv1 -. exposes .-> ServiceEnv1
        ServiceEnv1 -. routes .-> ExpositionEnv1
    end
    subgraph Env2VMInst["`Instanced Environment 2 _(VM-based)_`"]
        ExpositionEnv2["Ingress / HTTPRoute"]
        ServiceEnv2["Service"]
        VirtualMachine["VirtualMachine"]

        VirtualMachine -. exposes .-> ServiceEnv2
        ServiceEnv2 -. routes .-> ExpositionEnv2
    end
    VirtLauncherPod["KubeVirt VirtLauncher Pod"]

    VirtualMachine -- becomes --> VirtLauncherPod
    InstanceCR -- becomes --> Env1ContainerInst
    InstanceCR -- becomes --> Env2VMInst
end
GWAPI["Load Balancer + Gateway/Ingress"]

InstanceController -- reconciles --> InstanceCR
InstanceAutomationController -- reconciles --> InstanceCR
InstanceSnapshotController -- reconciles --> InstanceCR
ExpositionEnv2 -. routes .-> GWAPI
ExpositionEnv1 -. routes .-> GWAPI

classDef ControllerStyle stroke:#FF8000,rx:12,ry:12,fill:none
classDef CustomResourceStyle stroke:#FF0000,rx:12,ry:12,fill:none
classDef EnvironmentStyle stroke:#A865B5,rx:12,ry:12,fill:none
classDef WorkspaceNSStyle stroke:#0A6522,rx:12,ry:12,fill:none,stroke-dasharray:8
classDef TenantNSStyle stroke:#2222FF,rx:12,ry:12,fill:none,stroke-dasharray:8

class TenantNS TenantNSStyle
class Env1ContainerInst,Env2VMInst EnvironmentStyle
```