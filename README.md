# Quantum MRI Pulse Simulation
<center>
<img src="Therm_eq.png" alt="Thermal Equilibrium" width="1000">

<img src="Meas.png" alt="Measurement" width="1000">

```
# Define MRI Tissue Parameters (e.g., White Matter at 1.5T)
t1_tissue = 600.0  
t2_tissue = 80.0   

# Simulate Inversion Recovery over various TI (Inversion Times)
ti_values = np.linspace(0, 2500, 25) # 0 to 2500 ms
mz_measurements = []

simulator = AerSimulator()

for ti in ti_values:
    # Update noise model for this specific delay time
    current_noise_model = NoiseModel()
    error = thermal_relaxation_error(t1_tissue, t2_tissue, ti)
    current_noise_model.add_all_qubit_quantum_error(error, "delay")
    
    # Build Inversion Recovery Circuit: 180° -> Delay(TI) -> Measure Z
    qc = QuantumCircuit(1, 1)
    qc.rx(np.pi, 0)            # 180 degree inversion pulse
    qc.delay(ti, 0, unit='ms') # Wait for TI
    qc.measure(0, 0)
    
    # Execute
    compiled_circuit = transpile(qc, simulator)
    result = simulator.run(compiled_circuit, noise_model=current_noise_model, shots=2000).result()
    counts = result.get_counts()
    
    # Calculate Mz (Expectation value of Z)
    p0 = counts.get('0', 0) / 2000
    p1 = counts.get('1', 0) / 2000
    mz = p0 - p1 
    mz_measurements.append(mz)
```

<img src="T1.png" alt="T1 Relaxation" width="1000">
</center>
