---
layout: project
title: Torque Wrench
description: Mechanics of Materials Project
technologies: [MATLAB | Fusion 360 | ANSYS] 
image: /assets/images/Torque_wrench_pic.png
show_header_image: false 
---

# CAD Model
<hr class="section-divider">

<div class="image-row">
  <img src="{{ page.image | relative_url }}" class="torque-image">
  <img src="{{ '/assets/images/Torque_wrench_dimensions.png' | relative_url }}" class="torque-image">
</div>

*All dimensions are in inches



# Material
<hr class="section-divider">

The chosen material for the wrench was Ti-6Al-4V. One of the design requirements was that the material had to be a steel, aluminum, or titanium alloy. In general, steel has a high Young’s modulus and high yield strength, aluminum has a low Young’s modulus and low yield strength, and titanium has a low Young’s modulus but a high yield strength.

Materials with high yield strength performed well against the safety requirements (yield failure, crack growth, and fatigue stress). Materials with a low modulus performed well for the strain-gauge output requirement because they produce higher strain. The steel alloys tested tended to meet the safety requirements but failed the strain-gauge output requirement. In contrast, the aluminum alloys tended to fail the safety requirements but passed the strain-gauge output requirement. Since titanium alloys combine high yield strength with a low modulus, they satisfied both requirements.

| Material         | Young's Modulus (MPa) | Poisson's Ratio | Yield Strength (MPa) | KIC (MPa√m) | Fatigue Strength (MPa) |
|------------------|------------------------|------------------|-----------------------|--------------|--------------------------|
| Titanium 6Al-4V  | 16.50                  | 0.3405           | 122.00                | 99.10        | 102.45                   |
{: .material-table} 
*All values are taken from Granta EduPack 2025 R2. Minimum and maximum values were averaged to obtain the property values.

# Load & Boundry Conditions 
<hr class="section-divider">

The CAD model was divided into three separate bodies: the upper drive, the lower drive, and the handle (the lower drive represents the filleted region between the drive and the handle). The upper drive was given a clamped constraint by setting the displacement of all nodes in that body to zero. A force was applied at the end of the handlebar with a magnitude of 40 lbf, based on the rated torque of 600 in-lbf.

![Load & Boundry Conditions]({{ '/assets/images/Torque_wrench_load_boundry.png' | relative_url }}){: .torque-image}

# Normal Strain 
<hr class="section-divider">

The maximum normal strain occurred at the edge between the lower drive and the handle, with a value of 1.965 mε. The analysis also shows a normal strain of 1.164 mε at the strain gauge location.

<div class="image-row">
  <img src="{{ '/assets/images/Torque_wrench_strain.png'  | relative_url }}" class="torque-image">
  <img src="{{ '/assets/images/Torque_wrench_strain1.png' | relative_url }}" class="torque-image">
  <img src="{{ '/assets/images/Torque_wrench_strain2.png' | relative_url }}" class="torque-image">
</div>

# Prinicpal Stress
<hr class="section-divider">


<div class="image-row">
  <img src="{{ '/assets/images/Torque_wrench_principlestress.png'  | relative_url }}" class="torque-image">
  <img src="{{ '/assets/images/Torque_wrench_principlestress1.png' | relative_url }}" class="torque-image">
</div>

The maximum principle stress occurred at the edge between the lower drive and the handle, with a value of 93.11 ksi. The analysis shows a peak at the interface between the clamped region of the drive and the filleted region; however, this peak is caused by a singularity and is therefore disregarded.

# Load Point Deflection 
<hr class="section-divider">

The analysis showed a load-point deflection of 0.3370 in.

![Load & Boundry Conditions]({{ '/assets/images/Torque_wrench_deform.png' | relative_url }}){: .torque-image}

# FEM Calculation Results 
<hr class="section-divider">

| Maximum Normal Stress (ksi) | Strain at Gauge (mε) | Load-Point Deflection (in) |
|-----------------------------|------------------------|-----------------------------|
| 44.99                       | 1.164                  | 0.3370                      |
{: .material-table}
*Analysis was performed with a mesh element size of 0.125 in.

# Wrench Sensitivity 
<hr class="section-divider">

For a half-bridge strain-gauge configuration, the output (in mV/V) is approximately equal to the strain × 100. Based on the FEM analysis, the torque-wrench sensitivity (at the rated torque of 600 in-lbf) is 1.164 mV/V.

# Strain Gauges
<hr class="section-divider">

The strain gauges selected for this design are the [SGD-2/350-LY13 models from Dwyer Omega](https://www.dwyeromega.com/en-us/linear-strain-gages/SGD-LINEAR1-AXIS/p/SGD-2-350-LY13). Each gauge measures 7.6 mm (0.3 in) in length and 5.8 mm (0.24 in) in width. The gauges will be arranged in a half-bridge configuration. Based on the wrench dimensions, the clearance between the start of the lofted region and the center of the strain gauge is 0.8 in, and the handle thickness is 0.5 in. Given these constraints, the selected strain gauge is an appropriate size for the design.