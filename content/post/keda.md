---
title: "KEDA：事件驅動的 Kubernetes 自動擴展解決方案"
description: "KEDA 是 Kubernetes 的事件驅動自動擴展利器。本篇文章深入探討 KEDA 的核心功能、適用場景、配置範例，以滿足現代應用的擴展需求。"
summary: "KEDA 是 Kubernetes 的事件驅動自動擴展利器。本篇文章深入探討 KEDA 的核心功能、適用場景、配置範例，以滿足現代應用的擴展需求。"
featured_image: "/images/keda.jpg"
tags: [學習筆記,backend,cloud,心得分享]
date: 2024-12-29T16:58:33+08:00
toc: true
disable_share: true
---

## KEDA 簡介

KEDA 是一個 Kubernetes-based Event-driven 的 Autoscaler，白話文就是他可以基於不同的 events 來自動調整 Pod 的 Autoscaling。

簡單介紹幾個核心的功能：
- Event-driven：根據 Event 來決定 Autoscaling 的規模
- Vendor 相依性低：內建多種 Vendor 的 Autoscaler
- 元件構造：簡單精簡，分隔 Kubernetes 系統設計，非侵入式且支援多種 Workload

## 為何需要 KEDA？

我相信第一次接觸 KEDA 的同學們一定會有這個疑惑：Kubernetes 已經提供 HPA (HorizontalPodAutoscaler) 了，我們為何還需要 KEDA？  

主要的原因是：**HPA 提供的 Autoscaling 機制已經不足以支援複雜的擴展需求了**。  

眾所周知，HPA 開箱即用只提供 CPU / Memory 兩種 metrics，如果想使用 custom metrics / external metrics 的話就需要自己搞定 Adapter。  
但使用 KEDA，我們就可以直接對接大量的、各式各樣的 Event 來當作 Autosclaing 的 metrics，只能說一句真香。  
{{< box info >}}
其實 KEDA 最終還是需要創建 / 操作 HPA 來達到 Autoscaling 的目的，它實作了一個 external metrics 的 HPA，所以新建 KEDA resource 同時也可以看到對應的 HPA resource 被創建。(Ref: [Keda source code](https://github.com/kedacore/keda/blob/87158f3dae1d08ddb897079aa42d89c31fc4861c/controllers/keda/scaledobject_controller.go#L290-L291))
{{< /box >}}

請試想以下的情境：
- 固定時間的大量流量：美國/歐洲的上班時間，伴隨著大量的客戶起床開始使用，會產生大量的 traffic
- 搶票/促銷：這類典型的大流量高併發場景，短時間內可能需要應對數十倍的 traffic 增長
- Data pipeline：在一個 pipeline 的生命週期中，你可能會選擇一些 Message Queue 來當作解決方案，而這種情況你會希望用 Queue 中 Lag 的 metrics 來當作 Autoscaling 的基準

傳統 HPA 只能基於 CPU 或 Memory 的使用率進行擴展，可能無法即時反映 traffic 暴增情況。而 KEDA 可以透過各式各樣的 metrics 來動態的調整 Pod 數量，確保各種場景下使用者的體驗是流暢的。

## 架構

![image](https://i.imgur.com/LKVjpcW.png)
> Ref: [KEDA Concepts](https://keda.sh/docs/2.16/concepts/)

我會解釋架構圖中的幾個 components，以及 KEDA 觸發 Autoscaling 的流程，但實務上使用未必需要了解的這麼深入，我在最後會提供範例的 YAML。

### Components

#### ScaledObject
ScaledObject 是一個 Kubernetes CRD (Custom Resource Definition)，這個 CRD 包含以下資訊：
- 觸發條件 (Triggers)：例如外部系統的 metrics 或 events。
- 縮放行為 (Scaling Behaviors)：例如 Pod 的最小/最大 replica 數量。

#### Kubernetes API Server
ScaledObject 會與 Kubernetes API Server 溝通，將我們定義的 Triggers 和 Scaling Behaviors 註冊到 Kubernetes 中，讓 Kubernetes 知道如何通過 KEDA 進行 Autoscaling。

#### Metrics Adapter
將 external metrics (如來自 Prometheus、Kafka、AWS SQS 等) expose 為 Kubernetes 可用的指標，供 HPA 使用。

#### Horizontal Pod Autoscaler (HPA)
當 KEDA 需要擴展時，會通過 Metrics Adapter 提供 metrics，HPA 則會根據這些 metrics 調整 Pod 的 replica 數量。

#### External Trigger Source
這就是我們希望當作 Autoscaling 基準的外部 resource，如 Kafka、RabbitMQ、CloudWatch、Redis ... 等。

### 流程
1. Any Events?  
KEDA 通過 Scaler 監控 External Trigger Source，如果檢測到滿足條件的事件，則觸發 Autoscaling
2. Metrics Adapter  
External Trigger Source 通過 Metrics Adapter 轉換為 Kubernetes API 可用的 metrics，這些 metrics 被傳遞給 HPA。
3. HPA 調整 Pod  
HPA 根據提供的 metrics 進行計算，調整 Workload 中 Pod 的 replica 數量。
4. 持續監控變化  
當 event metrics 改變時，KEDA 持續更新數據並動態調整 Pod 的數量。

## Example
要使用 KEDA 很簡單，只需要直接定義一個 ScaledObject 的 Resource:
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: keda-scaleobject-poc
  namespace: poc
spec:
  minReplicaCount: 2
  maxReplicaCount: 5
  pollingInterval: 10 # Optional. Default: 30 seconds
  scaleTargetRef:
    name: <your workload name>
  triggers:
  - type: <your scaler>
    metadata:
      <base on your scaler>
```

填入你要 scale 的 target 和對應的 triggers 並部署到 Cluster 中，確認對應的 CRD 有被建立後，就可以開始監控他是否正常運行了

## 小結

從系統設計上，我會把 KEDA 理解為 HPA 和 external metrics 的中間層，把 metrics 的轉換和與 Kubernetes 的溝通這兩段抽離出來

使用 KEDA 時不需要知道和外部系統如何溝通，也不需要自己實作 Adapter。這些細節被封裝在 KEDA 中，有效解決了傳統 HPA 無法處理多元化和複雜擴展需求的痛點，降低了 Kubernetes 中基於外部 events 或是 metrics 來做 Autoscaling 的複雜度。

## Reference
1. https://keda.sh/docs/2.16/concepts/
2. https://github.com/kedacore/keda
3. https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/