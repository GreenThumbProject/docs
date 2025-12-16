# Optimal Conditions Prediction

Using machine learning to find optimal nutrient and environmental conditions for plant growth.

## Research Question

> "What are the optimal conditions for this plant species?"

**Input**: Environmental data + plant images  
**Output**: Optimized values for nutrients and environment

## Key Findings

### Nutrient Optimization in Aquaponics

Research on nutrient optimization showed remarkable results:

**Cost Reduction**:
- Feature selection reduced 20 chemical parameters to 5 key nutrients
- ~75% reduction in nutrient monitoring costs

**Growth Improvements**:

| Plant | Traditional Time | Optimized Time | Improvement |
|-------|------------------|----------------|-------------|
| Lettuce | 6-8 weeks | 4 weeks | ~50% faster |
| Kale | 12 weeks | 4.5 weeks | ~63% faster |

Lettuce grown with optimized nutrients reached 40-45 inches in diameter—significantly larger than soil-grown plants.

### Implications for GreenThumb

In a controlled DWC environment, these optimizations could be even more effective:

1. **Automated adjustments**: Microcontrollers can make real-time decisions based on data
2. **Precise control**: No external factors affecting nutrient delivery
3. **Continuous learning**: System improves recommendations over time

## Approach

1. Collect environmental and growth data over multiple cultivation cycles
2. Use ML to identify correlations between conditions and growth outcomes
3. Train models to recommend optimal settings for each growth stage
4. Implement feedback loop for continuous improvement

## Variables to Optimize

| Category | Variables |
|----------|-----------|
| Nutrients | N, P, K, Ca, Mg, pH, EC |
| Environment | Temperature, Humidity, Light intensity |
| Water | Temperature, Dissolved oxygen, Flow rate |

## References

- [1] Nutrient optimization for plant growth in Aquaponic irrigation using Machine Learning for small training datasets ([DOI](https://doi.org/10.1016/j.aiia.2022.05.001))
