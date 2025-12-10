---
layout: project
title: Torque Wrench
description: Mechanics of Materials Project
technologies: [MATLAB | Fusion 360 | ANSYS] 
image: /assets/images/Torque_wrench_pic.png
show_header_image: false 
---

# Overview
<hr class="section-divider">

The goal of this project is to design a non-ratcheting, 3/8-inch drive instrumented torque wrench rated for 600 in-lbf. The torque will be measured using strain gauges mounted on the sides of the wrench. The design objective is to maximize the wrench’s voltage output (mV/V) at the rated torque, and it must produce at least 1.0 mV/V at 600 in-lbf. The wrench must also meet several constraints: it must not fail under static loading, crack growth, or fatigue, material must be a steel, alunimum, or titantium alloy, and it must withstand a fully reversed torque of ±600 in-lbf for 10⁶ cycles.

| X₀ (Yield Failure) | Xₖ (Crack Growth) | Xₛ (Fatigue Stress) |
|--------------------|------------------|----------------------|
| 4.0                | 2.0              | 1.5                  |
{: .material-table}

# Base Design 
<hr class="section-divider">

A MATLAB script iterated through the handle thickness and the strain-gauge distance from the drive for different materials. The design used a square cross-section handlebar with a thick root section (0.8 × 0.8 in)—chosen so that none of the materials would fail at the root—and a necked section (h × b in). The root region ended 1 in before the strain gauge, and the overall handle length was held constant at 15 in. It is important to note that although the drawing above includes a lofted region connecting the root and necked sections of the handle (to reduce stress concentrations), the hand calculations assumed an abrupt transition between the two sections.

**Material Class MATLAB Script:** 

```matlab
classdef Material
    properties
        E            % Young's modulus (psi) 
        nu           % Poisson's ratio 
        su           % Tensile strength (psi) 
        KIC          % Fracture toughness (psi*sqrt(in)) 
        sfatigue     % Fatigue strength for 10^6 cycles (psi) 
        name         % Material name
    end
    methods
        function obj = Material(E, nu, su, KIC, sfatigue, name)
            obj.E = E;
            obj.nu = nu;
            obj.su = su;
            obj.KIC = KIC;
            obj.sfatigue = sfatigue;
            obj.name = name;
        end
    end
end
```
{: .code-scroll}

**Hand Calculations MATLAB Script:** 

```matlab
clear all; 
clc; 

% Global Parameters
M = 600;    % Max torque (in*lbf) 
a = 0.04;   % Crack depth (in) 

%% ========================= Testing =========================

L = 16;     % Length from drive to applied load (in) 
h = 0.75;   % Necked width (in) 
b = 0.5;    % necked thickness (in) 
C = 1;      % Distance from drive to strain gauge (in) 

h0 = 0.75;  % Root width (in) 
b0 = 0.5;   % Root thickness (in) 
L0 = L;     % Root Length (in) 

P = M / L;  % Applied Load 

% Define material and properties 
M42_Steel = Material(32e6, 0.29, 370e3, 15e3, 115e3, 'M42 Steel'); 

% Stress and deflection | Safety factor | Stain gauge 
[max_deflection, max_stress] = stress_and_deflection(M42_Steel, M, L, h, b, P, h0, b0, L0); 
[X_o, X_k, X_s] = safety_factor(M42_Steel, M, L, h, b, P, a, max_stress, h0, b0, L0); 
[strain_at_gauge, gauge_output] = strain_gauge(M42_Steel, M, L, h, b, P, C); 

% Display results
disp(['  ======================================== ' ...
       'Testing Results ' ...
       '========================================']);
T = table(max_deflection, max_stress, X_o, X_k, X_s, strain_at_gauge, gauge_output);
disp(T);

% max_deflection = 0.091 in | max_stress = 12.80 ksi 
% X_o = 28.9 | X_k = 2.95 | X_s = 8.98 
% strain at gauge = 375 microstrain | gauge+output = 0.38 mV/V 


%% ========================= Materials =========================

Steel_4140 = Material( ...
    mean([30.2 31.3]) * 1e6, ...    % E (psi)
    mean([0.285 0.295]), ...        % nu
    mean([86.3 104]) * 1e3, ...     % su (psi)
    mean([61.9 97.4]) * 1e3, ...    % KIC (psi*sqrt(in))
    mean([58.8 88.2]) * 1e3, ...    % fatigue strength (psi)
    'AISI 4140 Steel' );

Steel_4340 = Material( ...
    mean([29.7 30.9]) * 1e6, ...
    mean([0.285 0.295]), ...
    mean([112 138]) * 1e3, ...
    mean([51.9 87.4]) * 1e3, ...
    mean([69.5 103]) * 1e3, ...
    'AISI 4340 Steel' );

Steel_17_4PH = Material( ...
    mean([28.6 30]) * 1e6, ...
    mean([0.27 0.281]), ...
    mean([115 127]) * 1e3, ...
    mean([125 153]) * 1e3, ...
    mean([60.9 89.5]) * 1e3, ...
    '17-4PH Stainless Steel' );

Aluminum_7075_T6 = Material( ...
    mean([10 11]) * 1e6, ...
    mean([0.325 0.335]), ...
    mean([66.7 76.9]) * 1e3, ...
    mean([24.2 24.4]) * 1e3, ...
    mean([24.9 30.8]) * 1e3, ...
    'Aluminum 7075 T6' );

Aluminum_6061_T6 = Material( ...
    mean([9.66 10.2]) * 1e6, ...
    mean([0.325 0.335]), ...
    mean([34.8 40.6]) * 1e3, ...
    mean([27.3 32.8]) * 1e3, ...
    mean([17.4 23.8]) * 1e3, ...
    'Aluminum 6061 T6' );

Aluminum_2024_T3 = Material( ...
    mean([10.4 11]) * 1e6, ...
    mean([0.33 0.343]), ...
    mean([42.1 54]) * 1e3, ...
    mean([33.7 37.3]) * 1e3, ...
    mean([17.2 34]) * 1e3, ...
    'Aluminum 2024 T3' );

Titanium_6Al_4V = Material( ...
    mean([16.3 16.7]) * 1e6, ...
    mean([0.332 0.349]), ...
    mean([114 130]) * 1e3, ...
    mean([94.2 104]) * 1e3, ...
    mean([88.9 116]) * 1e3, ...
    'Titanium 6Al-4V' );

Titanium_6Al_4V_ELI = Material( ...
    mean([16 17]) * 1e6, ...
    mean([0.332 0.352]), ...
    mean([110 131]) * 1e3, ...
    mean([82.7 100]) * 1e3, ...
    mean([46.6 65.1]) * 1e3, ...
    'Titanium 6Al-4V ELI' );

Materials = [ ...
    Steel_4140, ...
    Steel_4340, ...
    Steel_17_4PH, ...
    Aluminum_7075_T6, ...
    Aluminum_6061_T6, ...
    Aluminum_2024_T3, ...
    Titanium_6Al_4V, ...
    Titanium_6Al_4V_ELI ...
];


%% ========================= Design Parameters & Dimensions =========================

% Values were chosen as no materials fail at root
h0 = 0.8; % Root width (in) 
b0 = 0.8; % Root thickness (in) 

L = 15;   % Length from drive to applied load (in) 

Distances = 1:1:(L-1); % Distance from drive to strain gauge (in) 
Widths = 0.1:0.1:0.8;    % Necked widths & thicknesses 


% Minimum Safety factors and strain gaige output
X_o_bench = 4;    
X_k_bench = 2; 
X_s_bench = 1.5; 
strain_bench = 0.001; 

P = M/L; % Applied load

% Loop through widths 
for iW = 1:length(Widths)

    h = Widths(iW); % Set necked width
    b = Widths(iW); % Set necked thickness 

    % Set up boolean array 
    nM = length(Materials);
    nC = length(Distances); 
    meets_safety = false(nC, nM); 
    
    % Loop through distances
    for iD = 1:length(Distances) 

        C = Distances(iD); % Set distance from drive to strain gauge
        L0 = C - 1; % Set root Length (end 1 in before strain gauge) (in) 

    
        % Loop through Materials
        for iM = 1:length(Materials)

            material = Materials(iM); % Set material

            % Stress and deflection | Safety factor | Stain gauge 
            [max_deflection, max_stress] = stress_and_deflection(material, M, L, h, b, P, h0, b0, L0);
            [strain_at_gauge, gauge_output] = strain_gauge(material, M, L, h, b, P, C);
            [X_o, X_k, X_s] = safety_factor(material, M, L, h, b, P, a, max_stress, h0, b0, L0);

            % Update boolean array 
            if X_o >= X_o_bench && X_k >= X_k_bench && X_s >= X_s_bench && gauge_output >= strain_bench
                meets_safety(iD, iM) = true; 
            else
                meets_safety(iD, iM) = false; 
            end
        end 
    end 
    
    % Create and display results table
    MaterialNames = string({Materials.name}); 
    safety_table = array2table(meets_safety, 'VariableNames', MaterialNames); 
    safety_table = addvars(safety_table, Distances','Before', 1, 'NewVariableNames', 'Distance (in)');
    fprintf(['\n  ================================================================= ' ...
        'Safety results for h = %.3f in (b = %.3f in) ' ...
        '=================================================================\n'], h, b);
    disp(safety_table);
end 

%% ========================= Selected Material =========================

% Using h = 0.5 in | b = 0.5 in | L = 15 in | C = 5 in | Titanium 6Al-4V

L = 15;     % Length from drive to applied load (in) 
h = 0.5;    % Necked width (in) 
b = h;      % Necked thickness (in) 
C = 5;      % Distance from drive to strain gauge (in) 
L0 = C - 1; % Root length (in) 

P = M / L;  % Applied Load 

% Stress and deflection | Safety factor | Stain gauge 
[max_deflection, max_stress] = stress_and_deflection(Titanium_6Al_4V, M, L, h, b, P, h0, b0, L0); 
[X_o, X_k, X_s] = safety_factor(Titanium_6Al_4V, M, L, h, b,P, a, max_stress, h0, b0, L0); 
[strain_at_gauge, gauge_output] = strain_gauge(Titanium_6Al_4V, M, L, h, b, P, C);

% Display results
T = table(max_deflection, max_stress, X_o, X_k, X_s, strain_at_gauge, gauge_output);
fprintf(['\n  = ' ...
        'Results for h = b = %.3f in | L = %.3f in | C = %.3f in | L0 = %.3f in | %s (Properties: E = %.3s psi, nu = %.3s) ' ...
        '=\n'], h, L, C, L0, Titanium_6Al_4V.name, Titanium_6Al_4V.E, Titanium_6Al_4V.nu);
disp(T);


%% ========================= Stress and Deflection =========================

function [max_deflection, max_stress] = stress_and_deflection(material, M, L, h, b, P, h0, b0, L0)
    % Extract material properties
    E = material.E;
    nu = material.nu;
    su = material.su;
    KIC = material.KIC;
    sfatigue = material.sfatigue;
    
    % Calculate stress and deflection
    I1 = b0*h0^3/12; % Root moment of inertia                         
    I2 = b*h^3/12;   % necked moment of inertia 

    M_neck = P*(L-L0); % Moment at neck region
    neck_stress = M_neck*(h/2)/I2;   % Stress at neck 
    root_stress = M*(h0/2)/I1;       % Stress at root 

    max_stress = max(root_stress, neck_stress); 

    % Energy method
    M = @(x) P.*(L-x);
    U = integral(@(x) M(x).^2/(2*E*I1), 0, L0) + integral(@(x) M(x).^2/(2*E*I2), L0, L);
    max_deflection = (2/P)*U; 
end 


%% ========================= Safety Factor =========================

function [X_o, X_k, X_s] = safety_factor(material, M, L, h, b, P, a, max_stress, h0, b0, L0)
    % Extract material properties
    E = material.E;
    nu = material.nu;
    su = material.su;
    KIC = material.KIC;
    sfatigue = material.sfatigue;

    % Calculate Safety factor for strength 
    X_o = su/max_stress; 

    % Calculate Safety factor for crack growth

    % For root
    Sg_root = 6*M/(b0*h0^2);
    KI_root = 1.12*Sg_root*sqrt(pi*a); 
    
    % For neck 
    M_neck = P*(L-L0); 
    Sg_neck = 6*M_neck/(b*h^2);
    KI_neck = 1.12*Sg_neck*sqrt(pi*a); 

    X_k = KIC/max(KI_root, KI_neck); 
    
    % Calculate Safety factor for fatigue
    X_s = sfatigue/max_stress; 
end


%% ========================= Strain Gauge =========================

function [strain_at_gauge, gauge_output] = strain_gauge(material, M, L, h, b, P, C)
    % Extract material properties
    E = material.E;
    nu = material.nu;
    su = material.su;
    KIC = material.KIC;
    sfatigue = material.sfatigue;
    
    % Calculate strain at gauge
    M_gauge = P*(L-C);
    I = b*h^3/12; 
    stress_gauge_1 = M_gauge*(h/2)/I; 
    strain_gauge_1 = stress_gauge_1/E;

    stress_gauge_2 = M_gauge*(-h/2)/I;
    strain_gauge_2 = stress_gauge_2/E;
    
    strain_at_gauge = strain_gauge_1; 

    % Calculate strain gauge output 
    k = 2; 
    gauge_output = (k/4)*(strain_gauge_1 - strain_gauge_2); 
end 
```
{: .code-scroll}


Based on these results, the titanium alloy Ti-6Al-4V was selected as the material. The necked region measures 0.5 × 0.5 in, and the strain gauge is positioned 5 in from the drive. 

The results of the hand calculations of the selected design are as follows: 

| Max Deflection (in) | Max Stress (Ksi) | X₀ | Xₖ | Xₛ | Strain at Gauge (ε) | Gauge Output (mV/V) |
|----------------------|------------------|----|----|----|----------------------|-----------------------|
| 0.2549               | 21.12            | 5.78 | 11.81 | 4.85 | 0.00116            | 1.16           |
{: .material-table}


# CAD Model
<hr class="section-divider">

<div class="image-row">
  <img src="{{ page.image | relative_url }}" class="torque-image">
  <img src="{{ 'assets/images/Torque_wrench_dimensions.png' | relative_url }}" class="torque-image">
</div>

*All dimensions are in inches



# Material
<hr class="section-divider">

The chosen material for the wrench was Ti-6Al-4V. One of the design requirements was that the material had to be a steel, aluminum, or titanium alloy. In general, steel has a high Young’s modulus and high yield strength, aluminum has a low Young’s modulus and low yield strength, and titanium has a low Young’s modulus but a high yield strength.

Materials with high yield strength performed well against the safety requirements (yield failure, crack growth, and fatigue stress). Materials with a low modulus performed well for the strain-gauge output requirement because they produce higher strain. The steel alloys tested tended to meet the safety requirements but failed the strain-gauge output requirement. In contrast, the aluminum alloys tended to fail the safety requirements but passed the strain-gauge output requirement. Since titanium alloys combine high yield strength with a low modulus, they satisfied both requirements.

| Material         | Young's Modulus (Ksi) | Poisson's Ratio | Yield Strength (Ksi) | KIC (Ksi√in) | Fatigue Strength (Ksi) |
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
  <img src="{{ 'assets/images/Torque_wrench_strain.png'  | relative_url }}" class="torque-image">
  <img src="{{ 'assets/images/Torque_wrench_strain1.png' | relative_url }}" class="torque-image">
  <img src="{{ 'assets/images/Torque_wrench_strain2.png' | relative_url }}" class="torque-image">
</div>

# Normal Stress
<hr class="section-divider">
The maximum principle stress occurred at the edge between the lower drive and the handle, with a value of 44.99 ksi. The analysis shows a peak at the interface between the clamped region of the drive and the filleted region; however, this peak is caused by a singularity and is therefore disregarded.

<div class="image-row">
  <img src="{{ 'assets/images/Torque_wrench_norm_stress.png'  | relative_url }}" class="torque-image">
  <img src="{{ 'assets/images/Torque_wrench_norm_stress1.png'  | relative_url }}" class="torque-image">
  <img src="{{ 'assets/images/Torque_wrench_norm_stress2.png' | relative_url }}" class="torque-image">
</div>

# Prinicpal Stress
<hr class="section-divider">

The maximum principle stress also occurred at the edge between the lower drive and the handle, with a value of 93.11 ksi. As formentioned, the analysis shows a peak at the interface between the clamped region of the drive and the filleted region; however, this peak is caused by a singularity and is therefore disregarded.

<div class="image-row">
  <img src="{{ 'assets/images/Torque_wrench_principlestress.png'  | relative_url }}" class="torque-image">
  <img src="{{ 'assets/images/Torque_wrench_principlestress2.png'  | relative_url }}" class="torque-image">
  <img src="{{ 'assets/images/Torque_wrench_principlestress1.png' | relative_url }}" class="torque-image">
</div>

# Load Point Deflection 
<hr class="section-divider">

The analysis showed a load-point deflection of 0.3370 in.

![Load & Boundry Conditions]({{ 'assets/images/Torque_wrench_deform.png' | relative_url }}){: .torque-image}

# FEM Calculation Results 
<hr class="section-divider">

| Maximum Normal Stress (ksi) | Strain at Gauge (mε) | Load-Point Deflection (in) |
|-----------------------------|------------------------|-----------------------------|
| 44.99                       | 1.164                  | 0.3370                      |
{: .material-table}
*Analysis was performed with a mesh element size of 0.125 in.

# Wrench Sensitivity 
<hr class="section-divider">

For a half-bridge strain-gauge configuration, the output (mV/V) is approximately equal to the strain × 1000. Based on the FEM analysis, the torque-wrench sensitivity (at the rated torque of 600 in-lbf) is 1.164 mV/V.

# Strain Gauges
<hr class="section-divider">

The strain gauges selected for this design are the [SGD-2/350-LY13 models from Dwyer Omega](https://www.dwyeromega.com/en-us/linear-strain-gages/SGD-LINEAR1-AXIS/p/SGD-2-350-LY13). Each gauge measures 7.6 mm (0.3 in) in length and 5.8 mm (0.24 in) in width. The gauges will be arranged in a half-bridge configuration. Based on the wrench dimensions, the clearance between the start of the lofted region and the center of the strain gauge is 0.8 in, and the handle thickness is 0.5 in. Given these constraints, the selected strain gauge is an appropriate size for the design.