# CoreDNS Timeout Debugging

Date: 2026-08-27

## Symptom

Intermittent `connection refused` / `i/o timeout` when resolving in-cluster DNS.

## Investigation

- Checked CoreDNS pod status: `kubectl -n kube-system get pods -l k8s-app=kube-dns`
- Found one replica restarting due to OOMKill.
- Looked at memory usage: `kubectl -n kube-system top pod`
- CoreDNS configmap was default, no custom upstream.

## Root cause

Node pressure on one worker; coredns pod exceeded memory limit during a burst.

## Mitigation

- Added `limits.memory` to 512Mi (was 170Mi) in CoreDNS deployment.
- Pinned CoreDNS to dedicated node via `nodeSelector` (optional).
- Monitored with `kubectl -n kube-system logs -l k8s-app=kube-dns --tail=20`

## Follow-up

- Consider NodeLocal DNSCache.
- Write alert for CoreDNS restarts.