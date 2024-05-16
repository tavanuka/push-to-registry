# Push to registry

A composite workflow to aid build and deployment of docker images to a container registry.

Provides a customizable workflow to build the desired repository into a docker image to be pushed to the desired registry.

The actions consists of primary docker actions to generate metadata and tags, and to build and push the image to the repository.

---

### Example

```yaml
# Example use
name: Example deployment

permissions:
  packages: write
  contents: write

on:
  push:

jobs:
  push-to-registry:
    runs-on: ubuntu-latest
    name: "Push image to registry"
    steps:
      - uses: tavanuka/push-to-registry@v1.0.0
        with:
          password: ${{ secrets.GITHUB_TOKEN }}
          publish: ${{ github.event.inputs.push_to_registry == 'true' }}
          registry: ghcr.io
          ref: ${{ github.event.inputs.version == '' && github.ref || format('refs/tags/{0}', github.event.inputs.version)  }}
          flavor: |
            latest=auto
          tags: |
            type=ref,event=tag,priority=1000
            type=raw,value=latest,priority=600,enable=${{ github.ref == format('refs/heads/{0}', 'main') }}
          annotations: |
            org.opencontainers.image.description=This is a test description
```

### Inputs

| Name          | Type    | Description                                                                                                             |
| ------------- | ------- | ----------------------------------------------------------------------------------------------------------------------- |
| `username`    | string  | The username of the container registry account. Default to repository owner. (default `github.repository_owner`)        |
| `password`    | string  | Password or a personal access token for the container registry account.                                                 |
| `publish`     | boolean | If true, it will publish the images to the specified registry.                                                          |
| `registry`    | string  | The container registry to publish the image to. (default `ghcr.io`)                                                     |
| `ref`         | string  | The reference branch, commit, or tag to push and version.                                                               |
| `images`      | string  | List of Docker images to use as base name for tags.                                                                     |
| `flavor`      | string  | [Flavor](https://github.com/docker/metadata-action?tab=readme-ov-file#flavor-input) to apply.                           |
| `tags`        | string  | List of custom [tags](https://github.com/docker/metadata-action?tab=readme-ov-file#tags-input).                         |
| `labels`      | string  | List of custom [labels](https://github.com/docker/metadata-action?tab=readme-ov-file#overwrite-labels-and-annotations). |
| `annotations` | string  | List of custom [annotations](https://github.com/docker/metadata-action?tab=readme-ov-file#annotations).                 |

### Outputs

| Name        | Type   | Description                                         |
| ----------- | ------ | --------------------------------------------------- |
| `image_tag` | string | The first tag out of the generated metadata output. |
