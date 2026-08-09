# External Route Created via CLI Not Showing in the RHOAI Dashboard

## This Is Expected Behavior: CLI-Created Routes Are Not Displayed in the RHOAI Dashboard

The RHOAI Dashboard manages external routes for model deployments through the built-in `odh-model-controller`. When you manually create a route using `oc create route` or `oc expose`, the resulting Route object lacks the specific labels, annotations, and naming conventions that the Dashboard relies on, so it will not be recognized or displayed.

> *"Manually exposing vLLM model endpoints, for example, by using `oc expose`, creates an unsecured route unless you configure authentication. Avoid exposing endpoints without security controls to prevent unauthorized access."*
>


## The Correct Approach: Enable External Routes Through the Dashboard

### Option 1: Enable during deployment (new deployments)

In the Deploy model wizard under **Advanced settings**, select the **"Model access"** checkbox. The `odh-model-controller` will automatically create a properly managed external route.

### Option 2: Edit an existing deployment

If the model is already deployed with only an internal endpoint:

1. Navigate to your model deployment in the Dashboard
2. Click **Edit** (edit deployment properties)
3. In the **Model route** section, select **"Make deployed models available through an external route"**
4. Save

The Dashboard will create the correct Route via `odh-model-controller`, and the external endpoint will appear on the deployment page.

Reference: [Editing the deployment properties of a model](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html-single/working_with_model_registries/index#editing-the-deployment-properties-of-a-model-deployed-by-using-the-model-serving-platform_model-registry)

## Why CLI-Created Routes Are Not Recognized

The Dashboard identifies routes based on the following conditions:

| Condition | Description |
|---|---|
| `opendatahub.io/dashboard: 'true'` label | The InferenceService must carry this label for the Dashboard to display it |
| Route naming convention | The Dashboard looks up routes in `routes.route.openshift.io` by model name |
| Managed by `odh-model-controller` | Routes must be created by the controller to have correct labels, annotations, and TLS configuration |

Manually created routes do not satisfy these conditions.

## Recommendation

Delete the manually created route, then enable the external route through the Dashboard by editing the existing deployment. This ensures the endpoint is correctly displayed in the Dashboard and is properly secured with TLS.

## Learn More

- [Deploying models on the model serving platform](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html-single/deploying_models/index#deploying-models-on-the-model-serving-platform_deploying-models) - includes the "Model access" checkbox for enabling external routes
- [Editing the deployment properties of a model deployed from Model Registry](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html-single/working_with_model_registries/index#editing-the-deployment-properties-of-a-model-deployed-by-using-the-model-serving-platform_model-registry) - includes the "Make deployed models available through an external route" checkbox
- [Managing RHOAI with GitOps (AI on OpenShift, community)](https://ai-on-openshift.io/odh-rhoai/gitops/) - "Model Serving" section explains why CLI/GitOps-created resources need the `opendatahub.io/dashboard: 'true'` label to appear in the Dashboard
