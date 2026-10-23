# HVAC_Climate: verification by analysis

> DUMMY EVIDENCE for the remotive_car COSA example; not a real analysis.

HVAC is a RemotiveTopology mock of body_can.dbc; HVACControl.Left/RightTemperature have the physical range 16 to 30 °C. The real ECU is required to saturate actuation at the range maximum; the mock follows the requested value.

Conclusion: the contracts of HVAC_Climate hold by design review of the RemotiveTopology configuration.
