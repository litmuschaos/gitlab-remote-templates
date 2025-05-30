# GitLab Remote Templates for LitmusChaos

In GitLab using the `include` keyword allows the inclusion of external YAML files. This helps to break down the CI/CD configuration into multiple files and increases readability for long configuration files. It's also possible to have template files stored in a central repository and projects include their configuration files. This helps avoid duplicated configuration, for example, global default variables for all projects.

`include` requires the external YAML file to have the extensions .yml or .yaml, otherwise the external file won't be included.

_`include` supports different inclusion methods like:_

<table style="width:100%">
  <tr>
    <th>Method</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>local</td>
    <td>Include a file from the local project repository.</td>
</tr>
<tr>
    <td>file</td>
    <td>Include a file from a different project repository</td>
</tr>
<tr>
    <td>remote</td>
    <td>Include a file from a remote URL. Must be publicly accessible</td>
</tr>
<tr>
    <td>template</td>
    <td>Include templates which are provided by GitLab.</td>
</tr>
</table>

## LitmusChaos Remote Templates

The templates in this repository can be used as `remote` template in the GitLab CI yaml where we can test the resiliency and performance of your application against different chaos experiments.

### Available Templates

This repository provides comprehensive chaos engineering templates for various experiments:

#### **Pod-Level Chaos Experiments**
- `container-kill-template.yml` - Kill containers in target pods
- `pod-delete-template.yml` - Delete target pods
- `pod-cpu-hog-template.yml` - Consume CPU resources in pods
- `pod-memory-hog-template.yml` - Consume memory resources in pods
- `pod-autoscaler-template.yml` - Test pod autoscaling behavior
- `disk-fill-template.yml` - Fill disk space in target pods

#### **Network Chaos Experiments**
- `pod-network-loss-template.yml` - Inject network packet loss
- `pod-network-latency-template.yml` - Inject network latency
- `pod-network-corruption-template.yml` - Inject network packet corruption
- `pod-network-duplication-template.yml` - Inject network packet duplication

#### **Node-Level Chaos Experiments**
- `node-cpu-hog-template.yml` - Consume CPU resources on nodes
- `node-memory-hog-template.yml` - Consume memory resources on nodes
- `node-io-stress-template.yml` - Create I/O stress on nodes

#### **Infrastructure Management**
- `install-litmus-template.yml` - Install LitmusChaos infrastructure
- `uninstall-litmus-template.yml` - Uninstall LitmusChaos infrastructure
- `all-experiment-template.yml` - Run multiple experiments in sequence

### How to Use the Remote Templates?

It is very simple to use these templates in your CI yaml. It can be done using `include:remote` in the GitLab CI YAML.

`include:remote` can be used to include a file from a different location, using HTTP/HTTPS, referenced by using the full URL. The remote file must be publicly accessible through a simple GET request as authentication schemas in the remote URL are not supported.

For example, including pod delete experiment using remote template:
```yaml
include:
  - remote: 'https://raw.githubusercontent.com/litmuschaos/gitlab-remote-templates/master/templates/pod-delete-template.yml'
```

You can provide multiple remotes in list format to add more templates:
```yaml
include:
  - remote: 'https://raw.githubusercontent.com/litmuschaos/gitlab-remote-templates/master/templates/pod-delete-template.yml'
  - remote: 'https://raw.githubusercontent.com/litmuschaos/gitlab-remote-templates/master/templates/container-kill-template.yml'
  - remote: 'https://raw.githubusercontent.com/litmuschaos/gitlab-remote-templates/master/templates/pod-cpu-hog-template.yml'
```

### Sample GitLab CI YAML using Pod Delete Remote Template

_gitlab-ci.yml_
```yaml
---
include:
  remote: 'https://raw.githubusercontent.com/litmuschaos/gitlab-remote-templates/master/templates/pod-delete-template.yml'

stages:
  - chaos

Inject Pod Delete Chaos:
  stage: chaos
  extends: .pod_delete_template
  variables:
    # LitmusChaos Configuration
    LITMUS_ENDPOINT: "https://your-litmus-endpoint.com"
    LITMUS_USERNAME: "admin"
    LITMUS_PASSWORD: "your-password"
    LITMUS_PROJECT_ID: "your-project-id"
    
    # Application Configuration
    APP_NS: "default"
    APP_LABEL: "app=nginx"
    APP_KIND: "deployment"
    
    # Experiment Configuration
    TOTAL_CHAOS_DURATION: "30"
    CHAOS_INTERVAL: "10"
    FORCE: "true"
```

### Configuration Variables

Each template supports various configuration variables:

#### **Common Variables (All Templates)**
- `LITMUS_ENDPOINT` - LitmusChaos control plane endpoint
- `LITMUS_USERNAME` - LitmusChaos username
- `LITMUS_PASSWORD` - LitmusChaos password
- `LITMUS_PROJECT_ID` - LitmusChaos project ID
- `APP_NS` - Target application namespace
- `APP_LABEL` - Target application label selector
- `APP_KIND` - Target application kind (deployment, statefulset, etc.)

#### **Infrastructure Variables**
- `INSTALL_INFRA` - Whether to install infrastructure (default: "true")
- `USE_EXISTING_INFRA` - Whether to use existing infrastructure (default: "false")
- `EXISTING_INFRA_ID` - ID of existing infrastructure (if using existing)
- `INFRA_NAME` - Name for new infrastructure
- `INFRA_NAMESPACE` - Namespace for infrastructure (default: "litmus")

#### **Experiment-Specific Variables**
Each template includes specific variables for its chaos experiment. Refer to individual template files for detailed configuration options.

### Multiple Experiments Example

```yaml
---
include:
  - remote: 'https://raw.githubusercontent.com/litmuschaos/gitlab-remote-templates/master/templates/pod-delete-template.yml'
  - remote: 'https://raw.githubusercontent.com/litmuschaos/gitlab-remote-templates/master/templates/pod-cpu-hog-template.yml'
  - remote: 'https://raw.githubusercontent.com/litmuschaos/gitlab-remote-templates/master/templates/pod-network-loss-template.yml'

stages:
  - chaos

variables:
  # Global LitmusChaos Configuration
  LITMUS_ENDPOINT: "https://your-litmus-endpoint.com"
  LITMUS_USERNAME: "admin"
  LITMUS_PASSWORD: "your-password"
  LITMUS_PROJECT_ID: "your-project-id"
  
  # Global Application Configuration
  APP_NS: "default"
  APP_LABEL: "app=nginx"
  APP_KIND: "deployment"

Pod Delete Chaos:
  stage: chaos
  extends: .pod_delete_template
  variables:
    TOTAL_CHAOS_DURATION: "30"

Pod CPU Hog Chaos:
  stage: chaos
  extends: .pod_cpu_hog_template
  variables:
    TOTAL_CHAOS_DURATION: "60"
    CPU_CORES: "2"

Pod Network Loss Chaos:
  stage: chaos
  extends: .pod_network_loss_template
  variables:
    TOTAL_CHAOS_DURATION: "60"
    NETWORK_PACKET_LOSS_PERCENTAGE: "50"
```

### Requirements

- LitmusChaos control plane deployed and accessible
- Target applications deployed in Kubernetes cluster
- Proper RBAC permissions for chaos experiments
- GitLab CI/CD environment with access to target cluster

### Support

For issues and questions related to these templates, please refer to the [LitmusChaos documentation](https://docs.litmuschaos.io/) or create an issue in this repository.


