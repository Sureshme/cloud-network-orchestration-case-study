# Multi-Cloud Network Orchestration Engine: Dynamic MTU & Jumbo Frames Capability

![Platform PM Case Study](https://img.shields.io/badge/Product_Management-Platform_%26_Infrastructure-blue?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Complete-success?style=for-the-badge)
[![Live Interactive Prototype](https://img.shields.io/badge/Demo-Live_Vercel_App-000000?style=for-the-badge&logo=vercel)](https://console-connect-jumbo-orchestration.vercel.app)

> **Platform Product Manager Case Study**  
> *Author:* Technical Product Manager | Product Owner — Network Automation, Console Connect  
> *Domain:* Network-as-a-Service (NaaS), Multi-Cloud Interconnects, Layer-2 Infrastructure Optimization

---

## 📊 Executive Summary

In enterprise Network-as-a-Service (NaaS) platforms, large-scale workloads (such as cross-region database replication and bulk telemetry transfers) require **Jumbo Frames (up to 9000 bytes MTU)** to minimize CPU overhead and maximize payload efficiency. 

However, enabling Jumbo Frames across hybrid multi-cloud topologies introduces significant operational complexity:
1. **Hardware Telemetry Mismatch:** Legacy access switches or specific Point of Presence (PoP) edge devices lack hardware support for >1500 MTU.
2. **Cloud Partner Variance:** Cloud providers enforce conflicting MTU policies—AWS Direct Connect supports up to 9001 MTU, whereas Microsoft Azure ExpressRoute gateways cap MTU at 1500.
3. **Silent Degradation Risk:** Forcing Jumbo Frames over an incompatible cloud path or Internet On-Demand (IOD) link causes silent packet fragmentation, elevated latency, and critical support overhead.

This project outlines the end-to-end product strategy, technical architecture, user experience guardrails, and dynamic monetization framework for an **Automated Network MTU Validation Engine**.

---

## 🤖 AI assisted Prototyping

Built using AI-assisted rapid prototyping (Claude for architecture/PRD drafting, Lovable for UI, Vercel for deployment) to compress requirements-to-prototype cycle time and de-risk technical trade-offs before engineering investment.

---

## 🎯 Strategic Outcomes & Business Impact

| Strategic Pillar | Capability Delivered | Business Impact |
| :--- | :--- | :--- |
| **Monetization & ARPU** | Dynamic pricing engine applying selective premiums to Jumbo-enabled ports/connections. | **Direct revenue expansion** on high-throughput enterprise connections. |
| **Customer Experience (UX)** | Context-aware UI guardrails and real-time validation warnings prior to connection provisioning. | **Elimination of silent packet drops** and reduction of post-provisioning MTU support tickets. |
| **Competitive Advantage** | Granular, per-service MTU selection across B2B, B2C, and IOD connections. | Benchmarked directly against competitors (e.g., Megaport, Equinix Fabric) to capture market share. |

---

## 🏗️ System Architecture & Validation Flow

The platform orchestration engine evaluates hardware capability and destination topology before permitting MTU configuration:

```text
+----------------------------------------------------------------------------------------+
|                                CUSTOMER & SYSTEM WORKFLOW                              |
+----------------------------------------------------------------------------------------+
  [Customer Selects Origin Port] 
                |
                v
  [Portal Queries Orchestrator API] 
                |
                v
  [Orchestrator Polls PoP Hardware Telemetry]
                |
                +---> Is Origin Port Jumbo Capable?
                          |
                          +--- NO  ---> [Force 1500 MTU - Hide Jumbo Toggle]
                          |
                          +--- YES ---> [Unlock Port Capability Badge]
                                                    |
                                                    v
                                      [Customer Selects Destination]
                                                    |
                                                    +---> Evaluate Target Endpoint Type
                                                              |
                                                              +--- AWS / Capable B2B  ---> [Unlock 9000/9001 MTU Toggle]
                                                              |                                        |
                                                              |                                        v
                                                              |                           [User Toggles Jumbo ON]
                                                              |                                        |
                                                              |                                        v
                                                              |                           [Billing Recalculates Rate + Premium]
                                                              |
                                                              +--- Azure / IOD / Legacy -> [Force 1500 MTU + Trigger UI Alert]
                                                                                                       |
                                                                                                       v
                                                                                          [Billing Applies Base Standard Rate]
                                                                                                       |
                                                                                                       v
                                                                                          [Provision Virtual Circuit]
