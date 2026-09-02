# 02. Product Requirements Document (PRD): Dynamic MTU & Jumbo Frames Orchestration

| Metadata | Details |
| :--- | :--- |
| **Document Status** | Approved for Development |
| **Product Capability** | Network Orchestration Layer (NaaS) |
| **Feature Name** | Automated MTU Validation & Dynamic Jumbo Frame Engine |
| **Target Launch** | Q3 2026 |

---

## 1. Problem & Objectives

### 1.1 Problem Statement
Enterprise users provisioning high-bandwidth Layer-2 links suffer from silent packet fragmentation and performance degradation when MTU sizes are misconfigured across heterogeneous network endpoints (PoP ports, AWS, Azure, IOD).

### 1.2 Core Objectives
* **Automate MTU Validation:** Dynamically evaluate origin port hardware capability, destination type, and MTU limits before circuit creation.
* **Protect Customer Experience:** Provide real-time UI/UX guardrails to prevent incompatible Jumbo Frame configurations (e.g., locking Azure or IOD to 1500 MTU with explicit warnings).
* **Enable Dynamic Monetization:** Integrate automated surcharge logic into the checkout billing engine when Jumbo Frames (9000/9001 MTU) are requested.

---

## 2. User Personas & Target Workloads

| Persona | Core Needs | Primary Pain Point |
| :--- | :--- | :--- |
| **Enterprise Network Architect** | High throughput for cross-cloud database replication and bulk storage sync. | Silent packet drops and poor latency due to mismatched cloud gateway MTUs. |
| **FinOps / Procurement Manager** | Clear, transparent line-item pricing before provisioning connections. | Unexpected billing line items or non-transparent port add-on costs. |
| **Platform Ops / Network Engineer** | Automated API-driven provisioning without manual validation scripts. | High ticket volume troubleshooting Layer-2 packet fragmentation issues. |

---

## 3. Detailed Functional Requirements

### FR-01: Origin Port Telemetry & Hardware Check
* **System Behavior:** When a user selects an origin port, the orchestration layer must query the switch telemetry service.
* **Validation:**
  * If `port_jumbo_capable == true` $\rightarrow$ Mark origin as Jumbo Ready.
  * If `port_jumbo_capable == false` $\rightarrow$ Force MTU to 1500, disable Jumbo Frame selection, and display info tooltip: *"Selected origin port switch does not support >1500 MTU."*

### FR-02: Multi-Cloud & Service Destination Validation Matrix

The platform must evaluate the destination type and enforce the following rules:

| Connection Type | Target Destination | MTU Selection Allowed | Max Allowed MTU | Default State | UI Action / Notice |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **B2B** | Secondary Port (Capable) | User Toggle | 9000 Bytes | Disabled (1500) | Show Toggle + Dynamic Pricing Premium |
| **B2B** | Secondary Port (Incapable) | Locked | 1500 Bytes | Disabled (1500) | Lock Toggle + Tooltip Notice |
| **B2C Cloud** | AWS Direct Connect | User Toggle | 9001 Bytes | Disabled (1500) | Show Toggle + Dynamic Pricing Premium |
| **B2C Cloud** | MS Azure ExpressRoute | Locked | 1500 Bytes | Disabled (1500) | Show Yellow Alert Callout + Lock Toggle |
| **B2C Cloud** | Google Cloud (GCP) | User Toggle (Restricted)| 1500 Bytes | Disabled (1500) | Informational Warning Modal |
| **Internet** | Internet On-Demand (IOD) | Locked | 1500 Bytes | Disabled (1500) | Lock Toggle + Public Transit Notice |

### FR-03: Dynamic Billing & Pricing Engine Integration
* **Pricing Rule:** If Jumbo Frames are toggled `ON`, the quote API must calculate:
  $$\text{Total Monthly Fee} = \text{Base Port Fee} + \text{Bandwidth Fee} + \text{Jumbo Surcharge}$$
* **Live Update:** The pricing drawer in the UI must update instantly without requiring a page refresh.
* **Audit Trail:** The final line-item invoice must record `jumbo_frames_enabled: true` and specify the effective MTU (e.g., `MTU: 9001`).

### FR-04: UI Guardrail Alert Specifications
* **Azure Warning Callout Text:**
  > ⚠️ **Cloud Provider Constraint:** *Microsoft Azure ExpressRoute gateways enforce a maximum payload limit of 1500 MTU. Enabling Jumbo Frames on this connection will cause packet fragmentation. This setting has been locked for reliability.*

---

## 4. User Stories & Acceptance Criteria

### User Story 1: Provisioning an AWS Cloud On-Ramp Link
* **As a** Network Architect,
* **I want to** select Jumbo Frames when connecting my origin port to AWS Direct Connect,
* **So that** I can optimize cross-cloud data replication throughput.
* **Acceptance Criteria:**
  1. System verifies origin port is Jumbo capable.
  2. UI unlocks the "Enable Jumbo Frames (9001 MTU)" toggle upon selecting AWS Direct Connect.
  3. Turning toggle ON increases the monthly quote breakdown by the configured premium (+15%).
  4. Provisioning request sends `effective_mtu: 9001` in the backend API payload.

### User Story 2: Preventing Misconfiguration on Azure
* **As a** Network Engineer,
* **I want to** be prevented from selecting Jumbo Frames on an Azure connection,
* **So that** my connection does not suffer silent packet drops.
* **Acceptance Criteria:**
  1. UI detects destination is MS Azure ExpressRoute.
  2. Toggle is explicitly locked to OFF (1500 MTU).
  3. Interactive alert box displays Azure MTU constraint details.
  4. Pricing premium is omitted from the quote summary.

---

## 5. Non-Functional Requirements (NFRs)

* **Performance:** Telemetry API check for port MTU capability must return in $< 150\text{ ms}$.
* **Reliability:** Validation rules engine must achieve $99.99\%$ uptime.
* **Audit & Compliance:** All MTU configuration state changes must be logged in audit trails with timestamp, user ID, and original port telemetry status.
