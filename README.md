# GitLab Remote Templates

In GitLab using the `include` keyword allows the inclusion of external YAML files. This helps to break down the CI/CD configuration into multiple files and increases readability for long configuration files. It’s also possible to have template files stored in a central repository and projects include their configuration files. This helps avoid duplicated configuration, for example, global default variables for all projects.

`include` requires the external YAML file to have the extensions .yml or .yaml, otherwise the external file won’t be included.

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

## Litmuschaos Remote Templates

The templates in this repository can be used as `remote` template in the GitLab CI yaml where we can test the resiliency and performance of your application against different chaos experiments.

### How to Use the remote template?

It is very simple to use these templates in your CI yaml. It can be done using `include:remote` in the GitLab CI YAML.
`include:remote` can be used to include a file from a different location, using HTTP/HTTPS, referenced by using the full URL. The remote file must be publicly accessible through a simple GET request as authentication schemas in the remote URL are not supported. 
For example including pod delete experiment using remote template:
```yaml
include:
  - remote: 'https://raw.githubusercontent.com/litmuschaos/gitlab-remote-templates/master/templates/pod-delete-template.yml'
```

We can provide multiple remotes in list formate to add more number of remote templates.

### A Sample GitLab CI YAML using pod delete remote template

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
  before_script:
  variables:
    APP_NS: default
    APP_LABEL: "app=nginx"
    APP_KIND: deployment
    TARGET_CONTAINER: nginx
```


