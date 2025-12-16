# Virtual Sensors

Using deep learning to create virtual sensors that reduce greenhouse hardware costs.

## Overview

Virtual sensors use machine learning to estimate measurements that would normally require physical sensors, reducing hardware costs and complexity.

> **Reference Paper**: "A Virtual Soil Moisture Sensor for Smart Farming Using Deep Learning" ([DOI: 10.1109/TIM.2022.3196446](https://doi.org/10.1109/TIM.2022.3196446))

## Concept

Instead of deploying expensive or hard-to-maintain sensors at every location, virtual sensors:

1. Use data from existing, cheaper sensors
2. Apply ML models to predict unmeasured variables
3. Reduce overall hardware costs significantly

## Application in Hydroponics

For GreenThumb, virtual sensors could estimate:

| Virtual Sensor | Based On | Potential Savings |
|----------------|----------|-------------------|
| Nutrient levels | EC + pH + time | Avoid frequent lab tests |
| Plant health status | Images + environment | Skip manual inspection |
| Root zone conditions | Surface measurements | Avoid invasive sensors |
| Evapotranspiration | Temp + humidity + light | Calculate water needs |

## Benefits

- **Cost Reduction**: Fewer physical sensors needed
- **Scalability**: Easy to add new virtual measurements
- **Fault Tolerance**: Continue operating if a physical sensor fails
- **Lower Maintenance**: Fewer sensors to calibrate and replace

## Implementation Approach

1. **Collect Training Data**: Gather paired readings from existing sensors and target measurements
2. **Train Models**: Use deep learning to learn relationships between inputs and outputs
3. **Deploy**: Run inference on the Raspberry Pi in real-time
4. **Validate**: Periodically compare with physical measurements

## Future Work

- [ ] Identify which sensors are candidates for virtualization
- [ ] Collect sufficient training data during PIBITI
- [ ] Experiment with model architectures (CNN, LSTM, etc.)
- [ ] Validate accuracy against physical sensors

## Related Articles

- [Optimal Conditions](optimal-conditions.md) - ML for nutrient optimization
- [Growth Prediction](growth-prediction.md) - Predicting plant development
