# ADR-0002: Use Proxmox and AWS for hosting

- Status: Accepted
- Date: 2026-11-26

## Context

This project needs a place to be hosted.

There is a budget constraint of $20/month.

There are 2 available Dell PowerEdge R715 servers running Proxmox at my remote lab.
The lab is reachable only over Tailscale.

## Decision

I chose to host this project at my remote lab to keep costs low.

I will use AWS for S3, a budget alarm, Lambda probe, and an
optional short-lived EKS test.

## Alternatives considered

I rejected hosting everything on AWS due to cost.

I also rejected running everything in one VM since I want to learn Kubernetes.

## Consequences

Operations get harder since it becomes a distributed application instead of staying
in one VM. Provisioning, networking, storage and debugging can span multiple VMs
on two physical hosts instead of one machine.

Everything is self managed which can be both good and bad.

Cost of electricity is high since the servers are old and less efficient.

