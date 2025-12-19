---
title: Helm Chart Values
sidebar_label: Helm Chart Values
---

# kthena

A Helm chart for deploying Kthena

![Version: 1.0.0](https://img.shields.io/badge/Version-1.0.0-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square) ![AppVersion: 1.0.0](https://img.shields.io/badge/AppVersion-1.0.0-informational?style=flat-square)

## Requirements

| Repository | Name | Version |
|------------|------|---------|
|  | networking | 1.0.0 |
|  | workload | 1.0.0 |

## Values

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| global.certManagementMode | string | `"auto"` |  |
| global.webhook.caBundle | string | `""` |  |
| networking.enabled | bool | `true` |  |
| networking.kthenaRouter.enabled | bool | `true` |  |
| networking.kthenaRouter.fairness.enabled | bool | `false` |  |
| networking.kthenaRouter.fairness.inputTokenWeight | float | `1` |  |
| networking.kthenaRouter.fairness.outputTokenWeight | float | `2` |  |
| networking.kthenaRouter.fairness.windowSize | string | `"1h"` |  |
| networking.kthenaRouter.gatewayAPI.enabled | bool | `false` |  |
| networking.kthenaRouter.gatewayAPI.inferenceExtension | bool | `false` |  |
| networking.kthenaRouter.image.pullPolicy | string | `"IfNotPresent"` |  |
| networking.kthenaRouter.image.repository | string | `"ghcr.io/volcano-sh/kthena-router"` |  |
| networking.kthenaRouter.image.tag | string | `"latest"` |  |
| networking.kthenaRouter.port | int | `8080` |  |
| networking.kthenaRouter.tls.dnsName | string | `"your-domain.com"` |  |
| networking.kthenaRouter.tls.enabled | bool | `false` |  |
| networking.kthenaRouter.tls.secretName | string | `"kthena-router-tls"` |  |
| networking.kthenaRouter.webhook.enabled | bool | `true` |  |
| networking.kthenaRouter.webhook.port | int | `8443` |  |
| networking.kthenaRouter.webhook.servicePort | int | `443` |  |
| networking.kthenaRouter.webhook.tls.certFile | string | `"/etc/tls/tls.crt"` |  |
| networking.kthenaRouter.webhook.tls.keyFile | string | `"/etc/tls/tls.key"` |  |
| networking.kthenaRouter.webhook.tls.secretName | string | `"kthena-router-webhook-certs"` |  |
| workload.controllerManager.downloaderImage.repository | string | `"ghcr.io/volcano-sh/downloader"` |  |
| workload.controllerManager.downloaderImage.tag | string | `"latest"` |  |
| workload.controllerManager.image.pullPolicy | string | `"IfNotPresent"` |  |
| workload.controllerManager.image.repository | string | `"ghcr.io/volcano-sh/kthena-controller-manager"` |  |
| workload.controllerManager.image.tag | string | `"latest"` |  |
| workload.controllerManager.runtimeImage.repository | string | `"ghcr.io/volcano-sh/runtime"` |  |
| workload.controllerManager.runtimeImage.tag | string | `"latest"` |  |
| workload.controllerManager.webhook.enabled | bool | `true` |  |
| workload.controllerManager.webhook.tls.certSecretName | string | `"kthena-controller-manager-webhook-certs"` |  |
| workload.controllerManager.webhook.tls.serviceName | string | `"kthena-controller-manager-webhook"` |  |
| workload.enabled | bool | `true` |  |