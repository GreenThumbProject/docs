# Growth Prediction Models

Predicting plant growth time using environmental data and computer vision.

> **Primary Reference**: Paper [3] - "A knowledge-and-data-driven modeling approach for simulating plant growth: A case study on tomato growth"

## Research Question

> "How long will the plant take to grow or develop fruits?"

**Input**: Environmental data + plant images  
**Output**: Time estimate (days, weeks, months)

## Key Approaches

### Knowledge-and-Data-Driven Model (KDDM) — Paper [3]

A hybrid approach combining domain knowledge with data-driven learning.

**Advantages**:

- **Scalable**: Easily add or remove input variables while maintaining accuracy
- **Robust**: Handles missing data effectively
- **Interpretable**: Preserves physically meaningful parameters

**Key Findings** (from tomato growth studies):

- Global radiation is the most important environmental variable
- Planting date significantly affects growth patterns
- Model can predict organ dry weights even when individual organ data is unavailable
- Performs better than pure mechanism-based models like GreenLab

### Visual Growth Prediction — Paper [5]

Predict future plant appearance based on current conditions.

**Input**: Current plant image + environmental data  
**Output**: Predicted plant image after X days

This enables:

- Growth timeline visualization
- Early detection of growth problems
- Harvest time optimization

## Dry Weight Prediction — Paper [3]

Predict total plant dry weight from environmental factors.

**Applications**:

- Yield estimation
- Harvest timing
- Resource optimization

The KDDM approach showed high accuracy for predicting dry weights of:

- Leaves (including petioles)
- Stems
- Fruits

## Key Variables

| Variable | Importance |
|----------|------------|
| Global radiation | Very High - Primary driver of growth |
| Planting date | High - Seasonal effects |
| Temperature | High - Growth rate modifier |
| Humidity | Medium - Transpiration effects |
| CO2 levels | Medium - Photosynthesis rate |

## References

- **[3]** A knowledge-and-data-driven modeling approach for simulating plant growth: A case study on tomato growth ([DOI](https://dx.doi.org/10.1016/j.ecolmodel.2015.06.006))
- **[5]** Plant Growth Prediction through Intelligent Embedded Sensing ([DOI](https://doi.org/10.1109/ISIE45063.2020.9152399))
