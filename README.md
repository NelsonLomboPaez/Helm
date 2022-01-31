# GitOps

Playing a bit with Red Hat GitOps Operator. Most of the information provided in this repository cames from the GitOps and Kubernetes book released by Manning, I'd recommend the book to anyone who wants to understand deeper GitOps, there are also a lot of information that is well explained in the [ArgoCD Upstream documentation](https://argo-cd.readthedocs.io/en/stable/) that it's very useful.


## Argo CD

Argo CD is an open source GitOps operator for Kubernetes, the code can be checked on the [argo cd repository]( https://argoproj.github.io/projects/argo-cd). The project is a part of the Argo family, a set of cloud-native tools for running and managing jobs and applications on Kubernetes. Along with Argo Workflows, Rollouts, and Events, Argo CD focuses on application delivery use cases and makes it easier to combine three modes of computing: services, workflows, and event-based processing.

### Core concepts(Applications and Projects)

#### Application

The Application CRD is the Kubernetes resource object representing a deployed application instance in an environment. It is defined by two key pieces of information:

- `source` reference to the desired state in Git (repository, revision, path, environment)
- `destination` reference to the target cluster and namespace. For the cluster one of server or name can be used, but not both (which will result in an error). Under the hood when the server is missing, it is calculated based on the name and used for any operations.

A minimal Application spec is as follows:

~~~
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: guestbook
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/argoproj/argocd-example-apps.git
    targetRevision: HEAD
    path: guestbook
  destination:
    server: https://kubernetes.default.svc
    namespace: guestbook
~~~
See [application.yaml](https://argo-cd.readthedocs.io/en/stable/operator-manual/application.yaml) for additional fields.

The Application provides a logical grouping of Kubernetes resources and defines a resources manifest’s source and destination. The Application source includes the repository URL and the directory inside of the repository. Typically repositories include multiple directories, one per application environment such as QA and Prod. The sample directory structure of such a repository is represented here:


~~~
├──  prod
│   └──  deployment.yaml
└──  qa
    └──  deployment.yaml
~~~

Each directory does not necessarily contain plain YAML files. Argo CD does not enforce any configuration management tool and instead provides first-class support for various config management tools. So the directory might as well contain a Helm chart definition as YAML along with Kustomize overlays

#### Project

The AppProject CRD is the Kubernetes resource object representing a logical grouping of applications. It is defined by the following key pieces of information:

- `sourceRepos` reference to the repositories that applications within the project can pull manifests from.
- `destinations` reference to clusters and namespaces that applications within the project can deploy into (don't use the name field, only the server field is matched).
- `roles` list of entities with definitions of their access to resources within the project.
- 
An example spec is as follows:

~~~
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: my-project
  namespace: argocd
  # Finalizer that ensures that project is not deleted until it is not referenced by any application
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  description: Example Project
  # Allow manifests to deploy from any Git repos
  sourceRepos:
  - '*'
  # Only permit applications to deploy to the guestbook namespace in the same cluster
  destinations:
  - namespace: guestbook
    server: https://kubernetes.default.svc
  # Deny all cluster-scoped resources from being created, except for Namespace
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  # Allow all namespaced-scoped resources to be created, except for ResourceQuota, LimitRange, NetworkPolicy
  namespaceResourceBlacklist:
  - group: ''
    kind: ResourceQuota
  - group: ''
    kind: LimitRange
  - group: ''
    kind: NetworkPolicy
  # Deny all namespaced-scoped resources from being created, except for Deployment and StatefulSet
  namespaceResourceWhitelist:
  - group: 'apps'
    kind: Deployment
  - group: 'apps'
    kind: StatefulSet
  roles:
  # A role which provides read-only access to all applications in the project
  - name: read-only
    description: Read-only privileges to my-project
    policies:
    - p, proj:my-project:read-only, applications, get, my-project/*, allow
    groups:
    - my-oidc-group
  # A role which provides sync privileges to only the guestbook-dev application, e.g. to provide
  # sync privileges to a CI system
  - name: ci-role
    description: Sync privileges for guestbook-dev
    policies:
    - p, proj:my-project:ci-role, applications, sync, my-project/guestbook-dev, allow
    # NOTE: JWT tokens can only be generated by the API server and the token is not persisted
    # anywhere by Argo CD. It can be prematurely revoked by removing the entry from this list.
    jwtTokens:
    - iat: 1535390316
~~~


# Red Hat GitOps Operator

The actual repository is wroten by using the Red Hat OpenShift GitOps operator 1.4.1 in the OpenShift 4.9 version, to check a newer version, please, refer to the newest documentation on [Red Hat GitOps site](https://docs.openshift.com/container-platform/4.9/cicd/gitops/gitops-release-notes.html).

## Installation process

There is a [Installing OpenShift GitOps operator in Red Had OpenShift documentation](https://docs.openshift.com/container-platform/4.9/cicd/gitops/installing-openshift-gitops.html) if the community version of the Argo CD Operator is installed in the cluster, it has to be removed before installing the Red Hat OpenShift GitOps Operator.

## Start playing

Now that the operator is installed on the cluster, let's play a bit with it, you can check some examples on the [following repository link]()
