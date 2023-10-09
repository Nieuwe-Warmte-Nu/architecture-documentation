# Requirements for frontend:
- Dynamically retrieves list of supported workflow types (defined dynamically and may change overtime)
- Keeps historical list of calculations
- ESDL is validated before submitting the job. ESDL is a correct heatnet ESDL and all input such as demands are attached. May still contain physics-related issues that can pop up during optimization.
	- May use the ESDL Validator for this (flask service) or handled by Frontend <-- Wouter and/or Stephan: After TPG deepdive, please update where this makes sense.