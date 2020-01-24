GPU Monitoring
==============

.. contents:: Table of Contents
   :local:


NVIDIA DCGM Exporter
--------------------

.. code-block:: sh

   # Check if the dcgm-exporter is successufully deployed
   $ kubectl get pods -n gpu-operator-resources -l app=nvidia-dcgm-exporter

   # Check gpu metrics locally
   $ dcgm_pod_ip=$(kubectl get pods -n gpu-operator-resources -o wide -l app=nvidia-dcgm-exporter | tail -n 1 | awk '{print $6}')
   $ curl $dcgm_pod_ip:9400/gpu/metrics


Deploying with Prometheus
-------------------------

.. code-block:: sh

   # To scrape gpu metrics from Prometheus, add dcgm endpoint to Prometheus via a configmap

   $ tee dcgmScrapeConfig.yaml <<EOF
   - job_name: gpu-metrics
     scrape_interval: 1s
     metrics_path: /gpu/metrics
     scheme: http

     kubernetes_sd_configs:
     - role: endpoints
       namespaces:
         names:
         - gpu-operator-monitoring

     relabel_configs:
     - source_labels: [__meta_kubernetes_pod_node_name]
       action: replace 
       target_label: kubernetes_node 
   EOF

   # Deploy Prometheus
   $ helm install --name prom-monitoring --set-file extraScrapeConfigs=./dcgmScrapeConfig.yaml stable/prometheus

   # Alternatively, if you find your prometheus pod pending and get this error "no persistent volumes available...", disable persistentVolumes. [Refer this](https://stackoverflow.com/questions/47235014/why-prometheus-pod-pending-after-setup-it-by-helm-in-kubernetes-cluster-on-ranch).
   $ helm install --name prom-monitoring --set-file extraScrapeConfigs=./dcgmScrapeConfig.yaml --set alertmanager.persistentVolume.enabled=false --set server.persistentVolume.enabled=false stable/prometheus

   # To check the metrics in browser
   $ kubectl port-forward $(kubectl get pods -lapp=prometheus -lcomponent=server -ojsonpath='{range .items[*]}{.metadata.name}{"\n"}{end}') 9090 &
   # Open in browser http://localhost:9090

