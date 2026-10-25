# Exterior lighting: breakdown of the informal requirements

This note justifies how the three top-level informal requirements of
ExteriorLightingDesign are refined over the realization, down to the atomic
components whose code or interface specification implements them. It is the
evidence cited by every `RationaleClaim_NonSafety_Composition` of the
exterior lighting.

| Id | Requirement | Status |
|---|---|---|
| EL-HAZ-1 | Hazard warning toggles on each press, overrides the turn stalk, resumes stalk side when off | reviewed |
| EL-RSP-1 | Indicator lamps respond within 200 ms, front and rear of a side at most 80 ms apart | reviewed |
| EL-BEAM-1 | Front lamps follow light mode and high beam; high-beam telltale | accepted, open finding |

## Design principle used throughout

Flashing and every decision about which side flashes are made **once**, in
LightingControl (BCM_Lighting). Every other building block only follows the
requests it receives. Because all followers consume the **same** CAN frame,
`TurnLightControl` on BodyCan0, the lamps and telltales stay in phase without
any synchronisation between FLCM, RLCM, DEVS2 and DIM. This is what lets each
top-level requirement split cleanly into one decision obligation and several
follow obligations.

```mermaid
flowchart LR
  sccm["LightingRequestSensing<br/>SCCM_Lighting<br/><i>reports raw inputs</i>"]
  bcm["LightingControl<br/>BCM_Lighting<br/><i>decides and flashes</i>"]
  flcm["FrontLighting<br/>FLCM_Lighting<br/><i>follows</i>"]
  rear["RearLighting<br/>RearLight<br/><i>follows</i>"]
  ind["LightingIndication<br/>LightingIndication<br/><i>follows</i>"]
  sccm -->|"DriverCan0<br/>TurnStalk, HazardLightButton, LightStalk"| bcm
  bcm -->|"BodyCan0<br/>one TurnLightControl frame"| flcm
  bcm --> rear
  bcm --> ind
```

## EL-HAZ-1 Hazard warning

```mermaid
flowchart TD
  top["EL-HAZ-1<br/>ExteriorLightingDesign"]
  top --> s["EL-HAZ-1.1<br/>SCCM_Lighting<br/>raw button and stalk state"]
  top --> c["EL-HAZ-1.2<br/>BCM_Lighting<br/>toggle, precedence, resume"]
  top --> f["EL-HAZ-1.3<br/>FLCM_Lighting<br/>front lamps follow"]
  top --> r["EL-HAZ-1.4<br/>RearLight<br/>rear lamps follow"]
  top --> i["EL-HAZ-1.5<br/>LightingIndication<br/>telltales follow, infotainment informed"]
  r --> r1["EL-HAZ-1.4.1<br/>RLCM_Lighting<br/>CAN to LIN unchanged"]
  r --> r2["EL-HAZ-1.4.2<br/>DEVS2_Lighting<br/>lamps follow ReqLeft/ReqRight"]
  i --> i1["EL-HAZ-1.5.1<br/>DIM_Lighting<br/>cluster telltales follow"]
  i --> i2["EL-HAZ-1.5.2<br/>GWM_Lighting<br/>TurnlightIndicator event"]
```

Why the children imply the parent:

- *Toggle on each press of at least 100 ms.* DriverCan0 samples the button
  every 50 ms (EL-HAZ-1.1), so a 100 ms press appears as 1 in at least one
  frame. BCM_Lighting toggles on the 0 to 1 edge (EL-HAZ-1.2), so a held
  button toggles once, not every cycle.
- *All four lamps and both telltales flash together.* BCM_Lighting sets both
  requests to the same value in the same frame (EL-HAZ-1.2); FLCM, RearLight
  and the DIM follow that frame without timing of their own (EL-HAZ-1.3,
  1.4, 1.5).
- *Resume on the stalk side.* BCM_Lighting keeps the last stalk position and
  returns to it on switch-off (EL-HAZ-1.2); SCCM keeps reporting the stalk
  while hazard is on (EL-HAZ-1.1).
- RL (RearLight) carries no obligation: it only answers RLCM's diagnostic
  counter and drives no turn lamp.
- IHU (LightingIndication) carries no obligation from EL-HAZ-1. The cluster
  telltale is the operating indication; the infotainment mirror is a
  convenience. EL-HAZ-1.5.2 only makes the state available to it. The
  IHU_Lighting event handler is a stub today, which this requirement does not
  depend on.

## EL-RSP-1 Response time and phase

The budget is derived and checked in `EL_Indicator_Response_Time_Analysis.md`.
Allocation to the application components follows the signal path; the parts
of each budget that the platform delivers (bus cycle, LIN schedule, ECU
scheduling) are allocated further to the platform components that the
existing Compute and Communicate allocations place the components on.

```mermaid
flowchart TD
  top["EL-RSP-1<br/>ExteriorLightingDesign<br/>≤ 200 ms, ≤ 80 ms apart"]
  top --> s["EL-RSP-1.1 SCCM_Lighting ≤ 50 ms"]
  top --> c["EL-RSP-1.2 BCM_Lighting ≤ 60 ms"]
  top --> f["EL-RSP-1.3 FLCM_Lighting 10–20 ms"]
  top --> r["EL-RSP-1.4 RearLight 10–65 ms"]
  r --> r1["EL-RSP-1.4.1 RLCM_Lighting ≤ 5 ms"]
  r --> r2["EL-RSP-1.4.2 DEVS2_Lighting 10–20 ms"]
  s -.->|CommunicateAllocation_DriverCan0| p1[("DriverCan0<br/>50 ms cycle")]
  c -.->|ComputeAllocation_BCM_Lighting| p2[("BCM_ECU<br/>≤ 10 ms reaction")]
  c -.->|CommunicateAllocation_BodyCan0| p3[("BodyCan0<br/>50 ms cycle")]
  r1 -.->|ComputeAllocation_RLCM_Lighting| p4[("RLCM_ECU<br/>≤ 5 ms reaction")]
  r1 -.->|CommunicateAllocation_RearLightLIN| p5[("RearLightLIN<br/>≤ 30 ms slot gap")]
```

## EL-BEAM-1 Light mode and high beam

```mermaid
flowchart TD
  top["EL-BEAM-1<br/>ExteriorLightingDesign"]
  top --> s["EL-BEAM-1.1 SCCM_Lighting<br/>assumption: switch steps through positions"]
  top --> c["EL-BEAM-1.2 BCM_Lighting<br/>mode table, high beam"]
  top --> f["EL-BEAM-1.3 FLCM_Lighting<br/>lamps follow, DRL dimmed with low beam"]
  top --> i["EL-BEAM-1.4 LightingIndication<br/>high-beam telltale"]
  i --> i1["EL-BEAM-1.4.1 DIM_Lighting"]
  s:::open
  classDef open stroke:#c0392b,stroke-width:2px
```

BCM_Lighting's beam state machine models a rotary switch: it only steps
Off ↔ DaylightRunningLight ↔ LowBeam. That design is correct only under the
assumption EL-BEAM-1.1 places on LightingRequestSensing. The breakdown made
the assumption explicit, and checking it against the SCCM mappings found that
the Logitech G29 mapping breaks it; see `../analysis/EL_Beam_Mode_Review.md`.
EL-BEAM-1 therefore stays *accepted* and is not argued as satisfied at
ExteriorLightingDesign.
