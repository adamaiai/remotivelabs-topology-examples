# Technical safety concept: allocation

> DUMMY EVIDENCE for the remotive_car COSA example; not a real work product.

Every software component runs on an ECU developed to at least its ASIL, with a compute budget of 5 MFLOPS out of the ECU's 50; every signal is carried by a bus whose application load stays below half its bit rate, so no allocation can starve a safety contract.
