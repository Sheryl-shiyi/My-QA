# Model Deployed with Custom Serving Runtime Not Appearing in Model Registry

## This Is Expected Behavior: Deployed Models Are Not Automatically Registered in Model Registry

In RHOAI, Model Registry and model deployment are completely decoupled workflows. Deploying a model (whether using a built-in or custom serving runtime) does not automatically register it in the Model Registry. The Model Registry is a metadata store that tracks model name, version, format, and storage location. You must register the model explicitly for it to appear.

## How to Register the Model in Model Registry

### Option 1: Register via the Dashboard UI

1. Navigate to **AI hub > Models > Registry**
2. Select your target model registry from the dropdown
3. Click **Register model**
4. Fill in the model details (name, version, source model format) and model location (S3 path or OCI URI)
5. Click **Register**

### Option 2: Register programmatically using the Python SDK

You can register models in a Pipeline or Notebook using the Model Registry Python SDK. The AI500 Lab includes examples of this approach.

## What Won't Happen Automatically: Linking the Existing Deployment

Registering the model metadata after deployment will not associate the already-running deployment with the new registry entry:

- The model version's **Deployments** tab in the Registry won't show your existing deployment, since it wasn't initiated from the Registry.
- RHOAI tracks this association via labels (`modelregistry/registered-model-id` and `modelregistry/model-version-id`) on the InferenceService CR, which are not present on directly deployed models.

## Recommendation

Register the model in the Registry, delete the current deployment, then redeploy from the Registry page using your custom serving runtime. This gives the Registry full visibility into deployment status.

Adopt a "register first, deploy from Registry" workflow to get complete model lifecycle management.

## Learn More

- [Registering a model](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html-single/working_with_model_registries/index#registering-a-model_model-registry) - steps for registering a model in the Model Registry
- [Adding a custom model-serving runtime](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html/configuring_your_model-serving_platform/configuring_model_servers#adding-a-custom-model-serving-runtime_rhoai-admin) - how to add custom serving runtimes
- [Deploying a model version from a model registry](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed/3.4/html-single/working_with_model_registries/index#deploying-a-model-version-from-a-model-registry_model-registry) - deploying from Registry with any serving runtime including custom ones
