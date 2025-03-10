# EnvChecker Installation Guide

## Table of Contents
- [Prerequisites](#prerequisites)
- [How to Configure View Role for Env-Checker](#how-to-configure-view-role-for-env-checker)
- [Third Party Software](#third-party-software)
- [Deployment](#deployment)
    - [Deployment Parameters](#deployment-parameters)
    - [HWE](#hwe)
- [Tests](#tests)
    - [Sanity Check](#sanity-check)
    - [Smoke Test Scenario](#smoke-test-scenario)

This document describes installation process for Qubership Environment Checker microservice.

## Prerequisites
Environment checker - should be installed inside the k8s cluster. The following are the prerequisites that must be met 
before you start with the installation.  The prerequisites are as follows:

It is required to perform a manual step of adding the clustered **view role** to the namespace account service (How to 
do this, see more in the chapter "How to configure view role for env-checker")

## How to configure view role for env-checker
The env-checker service is started as ServiceAccount=env-checker-sa. In order for queries to be executed inside the 
env-checker, you need to create a ClusterRoleBinding.

**ClusterRoleBinding.yaml example**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: view-for-env-checker
subjects:
  - kind: ServiceAccount
    name: env-checker-sa               # <--- env-checker service account
    namespace: {{ .Values.NAMESPACE }} # <--- fill current namespace
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: view                            # <--- ClusterRole with List of Rules for get\list\watch
```

## Third Party Software
The prerequisites are as follows:

| **Name**   | **Requirement** | **Version** |
|------------|-----------------|-------------|
| Kubernetes | Mandatory       | 1.21+       |

## Deployment

### Deployment Parameters
You may need to deploy the following Helm parameters during Environment checker installation. The deployment parameters 
are described in the following table.

| **Parameter**                            | **Is required?**        | **Default value**                                | **Value Example**                                                          | **Description**                                                                                                                                                                                                                          |
|------------------------------------------|-------------------------|--------------------------------------------------|----------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| CLOUD\_PUBLIC\_HOST                      | Mandatory               | -                                                | k8s-apps10.k8s.sdntest.qubership.org                                       | The public host is specified to create some Kuber elements, such as Ingress in env-checker.                                                                                                                                              |
| CHOWN_HOME                               | Mandatory for Openshift | -                                                | - / yes (for Openshift). possible values: 'yes' or 'no'                    | enables home directory ownership change during container deploy                                                                                                                                                                          | 
| CHOWN_HOME_OPTS                          | Mandatory for Openshift | -                                                | - / '-R' (for Openshift). possible values: - / '-R'                        | sets CHOWN_HOME mode to recursive                                                                                                                                                                                                        |  
| PRODUCTION\_MODE                         | Optional                | FALSE                                            | Possible values: TRUE or FALSE                                             | Flag indicating that the server is a production environment. env-checker will be launched in different modes (pod with Service/Ingres or no). How to change mode without reinstall check in guide: TBD                                   |
| ENVIRONMENT\_CHECKER\_LOG\_LEVEL         | Optional                | ERROR                                            | DEBUG                                                                      | Log level for all env-checker Notebooks. If the parameter was not specified explicitly then it will have the value = ERROR. The product uses only the ERROR logging level, but the parameter can be defined by any custom logging level. |
| OPS\_IDP\_URL                            | Optional                | -                                                | https://infra-keycloak-infra-keycloak.k8s-apps10.k8s.sdntest.qubership.org | URL to infra-keycloak. If IDP parameters are not defined then access to Env Checker is allowable via Jupiter default token **Required to IDP integration**                                                                               |
| ENVCHECKER\_KEYCLOACK\_REALM             | Optional                | -                                                | test-realm                                                                 | Name of IDP realm. User for Env-checker authentification have to belong to the realm **Required to IDP integration**                                                                                                                     |
| ENVCHECKER\_KEYCLOACK\_CLIENT\_ID        | Optional                | -                                                | test-env-checker                                                           | IDP Client ID. Client have to belong to the realm. Client parameter in IDP 'Valid Redirect URIs' have to contain URL to env-checker eg. https://env-checker-dev.k8s-apps10.k8s.sdntest.qubership.org **Required to IDP integration**     |
| ENVCHECKER\_KEYCLOACK\_CLIENT\_SECRET    | Optional                | -                                                | b4iwkh7nQBSxIgBEtlYSxUfNuoGZY19K                                           | IDP Client Secret. The value can be viewed in the Credentials tab on the idp client._If there is no Credentials tab. Set the Client authentication flag to ON_ **Required to IDP integration**                                           |
| ENVIRONMENT\_CHECKER\_JOB\_COMMAND       | Optional                | -                                                | ./run.sh notebooks/TestNotebook.ipynb                                      | Command to run env-checker shell in Job mode. **Required to create Kuber Job**                                                                                                                                                           |
| ENVIRONMENT\_CHECKER\_CRON\_JOB\_COMMAND | Optional                | -                                                | ./run.sh notebooks/TestNotebook.ipynb                                      | Command to run env-checker shell in CronJob mode. **Required to create Kuber CronJob**                                                                                                                                                   |
| ENVIRONMENT\_CHECKER\_CRON\_SCHEDULE     | Optional                | -                                                | 0 \*/1 \* \* \*                                                            | Schedule the release of CronJob in Cron format. Runs for non prod environments. **Required to create Kuber CronJob**                                                                                                                     |
| ENVIRONMENT_CHECKER_UI_ACCESS_TOKEN      | Optional                | f9a3bd4e9f2c3be01cd629154cfb224c2703181e050254b5 | token12345                                                                 | Token to log in to Env-Checker UI.                                                                                                                                                                                                       |

### HWE
All information about profiles and the amount of allocated resources for them can be found at the
[following link](HardwareEstimationAndSizing.md):

## Tests
### Sanity check
Check for non-prod environments

1) Log in to Kubernetes
2) Go to the namespace where env-checker was installed
3) Go to ingress service link (token to log in to UI may be configured via optional Helm parameter 
ENVIRONMENT_CHECKER_UI_ACCESS_TOKEN. If this parameter is not set, **"f9a3bd4e9f2c3be01cd629154cfb224c2703181e050254b5"** 
must be used to log in)
4) **ER** - Check if UI env-checker is available

### Smoke test scenario
For a smoke test, firstly need to make sure that the prerequisites have been set correctly, namely the access rights for the service account. To do this, run the command in the JupiterLab UI terminal:
```
kubectl get ns
```
If an error occurs, check that the ClusterRoleBinding was created correctly