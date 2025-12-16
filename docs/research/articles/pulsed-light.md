# Pulsed Light Techniques

Optimizing energy use and photosynthetic response using pulsed LED lighting.

## Overview

Research shows that continuous lighting can be substituted by pulsed light with comparable or improved photosynthetic efficiency—potentially reducing energy costs.

## Key Findings

### Photosynthesis with Pulsed Light

Studies on tomato plants showed:

- **2ms light / 198ms dark pulses**: Reduced photosynthetic rate to ~50% compared to continuous light
- **1 Hz pulsed light**: Produced best quantum efficiency values
- **750 µmol/m²/s intensity**: Outperformed 450 µmol/m²/s in quantum efficiency

!!! note "Light/Dark Synchronization"
    Proper timing between light and dark periods is critical. The plant must synchronize with the pulse frequency to optimize harvested light use and reduce heat loss.

### Quantum Efficiency

| Light Type | Quantum Efficiency (ΦPSII) |
|------------|---------------------------|
| Continuous | Baseline |
| 1 Hz Pulsed | Best performance |
| Higher frequencies | Diminishing returns |

The Electron Transport Rate (ETR) was also highest at 1 Hz pulsed light.

## Benefits

### Energy Savings

- Reduced electricity consumption with optimized pulse timing
- LED lifetime extension through reduced thermal stress
- Lower cooling requirements

### Plant Response

- Improved photochemical efficiency of photosystem II (ΦPSII)
- Comparable or better growth than continuous light
- Potential for tailored lighting recipes per species

## Implementation Considerations

For GreenThumb:

1. **LED Controller**: PWM control for pulse frequency adjustment
2. **Testing**: Compare continuous vs various pulse frequencies
3. **Monitoring**: Track growth metrics under different light regimes
4. **Species-Specific**: Different plants may respond differently

## Research Applications

This technique could enable:

- **Reduced operating costs** for indoor farming
- **Optimized light recipes** for each growth stage
- **Research into photosynthesis efficiency**

## References

- [4] A LED-based smart illumination system for studying plant growth ([DOI](https://doi.org/10.1177/1477153513478300))
- [10] Steinitz B, Poff KL. A single positive phototropic response induced with pulsed light in hypocotyls of Arabidopsis thaliana seedlings. Planta 1986
- [12] Tennessen DJ, et al. Efficiency of photosynthesis in continuous and pulsed light emitting diode irradiation. Photosynthesis Research 1995
