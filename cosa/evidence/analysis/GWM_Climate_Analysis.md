# GWM_Climate: verification by analysis

> DUMMY EVIDENCE for the remotive_car COSA example; not a real analysis.

on_hvac_control copies LeftTemperature and RightTemperature from the HVACService event into HVACControl unchanged (float conversion only), so GWM never raises a setpoint. test_hvac.py covers the left side only. Side finding outside this goal: a missing parameter is sent as 0 °C (float(... or 0)), below the 16 °C DBC minimum.

Conclusion: the contracts of GWM_Climate hold by design review of the RemotiveTopology configuration.
