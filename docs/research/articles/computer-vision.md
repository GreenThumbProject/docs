# Computer Vision Models

Using computer vision for plant health monitoring, growth analysis, and phenotyping.

## Applications

### Plant Health Detection

Detect nutrient deficiencies and diseases through visual analysis.

**Approach**:
- Analyze leaf color variations
- Detect spots, yellowing, or wilting
- Identify patterns associated with specific deficiencies

**Benefits**:
- Early detection before visible symptoms worsen
- Non-invasive monitoring
- Continuous automated surveillance

### Growth Visualization

Predict future plant appearance for growth tracking.

**Input**: Current plant image + environmental conditions  
**Output**: Predicted appearance after specified time

**Use Cases**:
- Visualize growth trajectory
- Identify plants falling behind expected growth
- Compare actual vs predicted development

### Leaf Area Estimation

Non-invasive measurement of plant leaf area.

**Method**:
- Reference squares with known areas in the image
- Machine vision calculates leaf area in absolute values
- Accounts for camera displacement

!!! info "Key Requirement"
    Reference squares are essential for obtaining accurate area measurements and reducing the impact of camera position changes.

### Dry Weight Estimation

Estimate plant dry weight from visual features.

**Advantages over traditional methods**:
- Non-destructive (no need to uproot or cut plants)
- Lower cost than multispectral cameras or tomographs
- Continuous monitoring possible

## Technical Considerations

### Cost-Effective Approach

Most phenotyping solutions are either:
- **Invasive**: Require cutting or uprooting plants
- **Expensive**: Multispectral cameras, tomographs

Machine vision with standard cameras offers:
- Non-invasive measurement
- Affordable hardware
- Adequate accuracy for most applications

### Implementation Notes

For GreenThumb:
1. USB camera already captures images every 4 hours
2. Add reference markers for area calibration
3. Process images with OpenCV for feature extraction
4. Train models on collected dataset

## Future Work

- [ ] Plant detection in images
- [ ] Leaf segmentation
- [ ] Color analysis for health monitoring
- [ ] Growth rate calculation

## References

- [5] Plant Growth Prediction through Intelligent Embedded Sensing ([DOI](https://doi.org/10.1109/ISIE45063.2020.9152399))
- [6] Vision-Based Object Detection for Efficient Monitoring in Smart Hydroponic Systems ([DOI](https://doi.org/10.2991/978-94-6463-364-1_40))
