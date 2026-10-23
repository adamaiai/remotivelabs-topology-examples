# IHU_Climate: verification by analysis

> DUMMY EVIDENCE for the remotive_car COSA example; not a real analysis.

The Android emulator's VHAL configures HVAC_TEMPERATURE_SET with a range of 16 to 28 °C, so the UI cannot request more than 28 °C; broker_to_emulator.py forwards the VHAL value to HVACService unchanged. No test requests a setpoint above the range.

Conclusion: the contracts of IHU_Climate hold by design review of the RemotiveTopology configuration.
