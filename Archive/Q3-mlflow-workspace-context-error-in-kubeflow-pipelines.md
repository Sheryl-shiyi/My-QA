# MLflow "Workspace Context" Error in Kubeflow Pipelines

## The Problem

MLflow integration works correctly in Jupyter workbenches but fails with a "workspace context" error when running Kubeflow pipelines. The RHOAI documentation lists a workaround for this error, but it does not resolve the issue in the pipeline context.

## Root Cause Analysis

This issue likely involves two related known issues that compound in a pipeline environment.

### Known Issue 1: RHOAIENG-67534 - "Workspace context is required for this request"

If the MLflow custom resource (CR) is created after the Evaluations CR, the `workspaces_enabled` setting is set to `false`. As a result, requests fail with an `INVALID_PARAMETER_VALUE` error: "Workspace context is required for this request."

**Documented workaround:** Create the MLflow CR before the Evaluations CR to ensure `workspaces_enabled` is correctly set to `true`.

This workaround targets evaluation runs initiated from the RHOAI Dashboard. It may not fully resolve the issue in a Kubeflow pipeline context, because pipeline tasks run as separate pods with a different authentication and configuration environment.

### Known Issue 2: RHOAIENG-44516 - MLflow tracking server does not accept Kubernetes service account tokens

When authenticating through the dashboard MLflow URL without using the workbench MLflow integration, RHOAI does not accept Kubernetes service account tokens.

This explains why the same code works in a workbench but fails in a pipeline:

- **In a workbench:** The `opendatahub.io/mlflow-instance` annotation automatically configures MLflow integration, including the `MLFLOW_TRACKING_AUTH` environment variable for service account token authentication.
- **In a pipeline:** Pipeline tasks run as independent pods without the workbench's automatic MLflow integration. The pod authenticates with its Kubernetes service account token, which the MLflow gateway does not accept.

**Documented workaround:** Create an OpenShift Route directly to the MLflow service endpoints and use the Route URL as the `MLFLOW_TRACKING_URI` when authenticating, bypassing the gateway.

## Recommendation

1. **Verify `workspaces_enabled` is set to `true`** by ensuring the MLflow CR was created before the Evaluations CR. If not, recreate the MLflow CR in the correct order.
2. **For pipeline authentication**, create an OpenShift Route directly to the MLflow service endpoints and configure pipeline tasks to use that Route URL as `MLFLOW_TRACKING_URI`, bypassing the RHOAI gateway.
3. If both workarounds are applied and the issue persists, consider opening a support case referencing both known issues, as the combination of MLflow + Kubeflow pipelines may require additional configuration not yet covered in the documentation.

## Learn More

- [RHOAIENG-67534 - Workspace context error (Known Issues)](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.5/html-single/release_notes/index#known-issues_RHOAIENG-67534_relnotes) - `workspaces_enabled` set to `false` when MLflow CR is created after Evaluations CR
- [RHOAIENG-44516 - MLflow tracking server does not accept Kubernetes service account tokens (Known Issues)](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html-single/release_notes/index#known-issues_RHOAIENG-44516_2_relnotes) - service account tokens rejected when not using workbench integration

