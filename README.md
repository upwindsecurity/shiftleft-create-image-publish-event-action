# Upwind Security ShiftLeft Create Image Publish Event Action

## Overview

The Upwind Security ShiftLeft Create Image Publish Event Action notifies the Upwind Console that a Docker image has been published, associating the published tag with the image. Use it after an image has been pushed to a registry so that Upwind can track which tag corresponds to which image.

Under the hood it downloads the `shiftleft` binary from the Upwind release bucket (authenticating with your Upwind credentials) and runs its `event` subcommand with `--event-type=IMAGE_PUBLISH`.

This action is the partner to the [`shiftleft-create-image-scan-event-action`](https://github.com/upwindsecurity/shiftleft-create-image-scan-event-action). The scan action scans a built image for vulnerabilities, while this action records that the image has been published. They are typically used together — scan on build, then publish on release.

## Prerequisites
- Supported runner architectures: `linux/amd64` and `linux/arm64`.
- Docker Environment: Ensure the GitHub runner has access to Docker to manage and reference images.
- Upwind Credentials: Obtain your Upwind Client ID and Client Secret for authentication.

## Inputs

Define the following inputs in your workflow to configure the action:

| Input                   | Required | Default     | Description                                                                                          |
|-------------------------|----------|-------------|------------------------------------------------------------------------------------------------------|
| `upwind_client_id`      | Yes      | –           | Your Upwind Client ID.                                                                               |
| `upwind_client_secret`  | Yes      | –           | Your Upwind Client Secret.                                                                           |
| `docker_image`          | Yes      | –           | The published Docker image (including tag), residing on the same runner or referenced by its full registry name. |
| `docker_user`           | No       | –           | Username for authenticating to the Docker registry.                                                 |
| `docker_password`       | No       | –           | Password for authenticating to the Docker registry.                                                 |
| `pull_image`            | No       | `true`      | Whether to pull the image. Set to `false` if the image is already available locally.                |
| `upwind_uri`            | No       | `upwind.io` | Public Upwind URI domain name.                                                                      |
| `additional_registries` | No       | –           | Comma-separated list of additional registries to associate with the published image.               |
| `debug`                 | No       | `false`     | Enable debug logging.                                                                               |

Sensitive values such as `upwind_client_id`, `upwind_client_secret`, and `docker_password` should be stored securely using GitHub Secrets.

## Usage

To integrate the image publish event into your GitHub workflow, include the following step:

```yaml
- name: Upwind Security ShiftLeft Image Publish
  uses: upwindsecurity/shiftleft-create-image-publish-event-action@main
  with:
    upwind_client_id: ${{ secrets.UPWIND_CLIENT_ID }}
    upwind_client_secret: ${{ secrets.UPWIND_CLIENT_SECRET }}
    docker_image: 'your-docker-image:tag'
    docker_user: ${{ secrets.DOCKER_USER }}
    docker_password: ${{ secrets.DOCKER_PASSWORD }}
    pull_image: false
    additional_registries: 'registry1,registry2'
```

## Versioning
It is recommended that you track the `main` branch rather than a specified tag. This will ensure that you always have the most up to date version of the action.

## Example Workflow

Below is a sample GitHub Actions workflow that builds a Docker image, scans it with the partner scan action, and then sends a publish event for the released tag:

```yaml
name: Docker Image Build, Scan and Publish

on:
  push:
    branches:
      - main

jobs:
  build-scan-publish:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout Code
        uses: actions/checkout@v6

      - name: Set Up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Build Docker Image
        run: |
          docker build . -t your-docker-image:${GITHUB_SHA}

      - name: Upwind Security ShiftLeft Scan
        uses: upwindsecurity/shiftleft-create-image-scan-event-action@main
        with:
          upwind_client_id: ${{ secrets.UPWIND_CLIENT_ID }}
          upwind_client_secret: ${{ secrets.UPWIND_CLIENT_SECRET }}
          docker_image: 'your-docker-image:${GITHUB_SHA}'
          pull_image: false

      - name: Upwind Security ShiftLeft Image Publish
        uses: upwindsecurity/shiftleft-create-image-publish-event-action@main
        with:
          upwind_client_id: ${{ secrets.UPWIND_CLIENT_ID }}
          upwind_client_secret: ${{ secrets.UPWIND_CLIENT_SECRET }}
          docker_image: 'your-docker-image:your-released-version'
          pull_image: false
```

This workflow triggers on pushes to the `main` branch, builds the Docker image, scans it for vulnerabilities, and then records the published tag with Upwind. The image does not need to be pulled because it is available locally via the Docker daemon.

## Troubleshooting
- Authentication Issues: Verify that your Upwind credentials are correct and have the necessary permissions.
- Docker Access: Ensure that the GitHub runner has the required permissions to access Docker.
