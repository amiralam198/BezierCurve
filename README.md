# Interactive Bézier Curve with Physics & Sensor Control

# Overview

This project implements an interactive cubic Bézier curve that behaves like a springy rope and would respond in real-time to mouse/touch input. All mathematical computations, physics simulations, and rendering are implemented from scratch without using any prebuilt animation or Bézier libraries.

# Mathematical Implementation

1. Cubic Bézier Curve Formula

The curve is generated using the standard cubic Bezier formula with four control points (P₀,P₁,P₂,P₃):

B(t) = (1−t)³P₀ + 3(1−t)²tP₁ + 3(1−t)t²P₂ + t³P₃


Where `t ∈ [0,1]` is the parametric variable.

**Implementation Details:** 
- The curve is sampled at `t` increments of 0.01 (100 points) for smooth rendering

- Coefficients are computed as:
   `c₀ = (1−t)³`
   `c₁ = 3(1−t)²t`
   `c₂ = 3(1−t)t²`
   `c₃ = t³`
- Each point is calculated by the weighted sum: `c₀P₀ + c₁P₁ + c₂P₂ + c₃P₃`

### 2. Tangent Vector Calculation

Tangent vectors are computed using the first derivative of the Bézier curve:

```
B'(t) = 3(1−t)²(P₁−P₀) + 6(1−t)t(P₂−P₁) + 3t²(P₃−P₂)
```

**Implementation Details:**
- Tangents are calculated at intervals of `t = 0.08` along the curve
- Each tangent vector is normalized to unit length
- Tangent lines are drawn with length 24 pixels centered at each sample point
- Color: semi-transparent yellow (`rgba(255,240,180,0.95)`) for visibility

## Physics Model

### Spring-Damping System

The dynamic control points (P₁ and P₂) use a spring-damper physics model to create smooth, natural motion:

```
acceleration = (-k × (position - target) - damping × velocity) / mass
```

**Parameters:**
- **Spring constant (k):** 150 — controls stiffness/responsiveness (tuned for optimal response)
- **Damping coefficient:** 18 — prevents oscillation and creates smooth settling
- **Mass:** 1 — normalized for simplicity

**Integration Method:**
- Semi-implicit Euler integration for numerical stability
- Update sequence:
  1. Compute spring force: `F_spring = -k(position - target)`
  2. Compute damping force: `F_damping = -damping × velocity`
  3. Calculate acceleration: `a = (F_spring + F_damping) / mass`
  4. Update velocity: `velocity += acceleration × dt`
  5. Update position: `position += velocity × dt`

This creates a "rope-like" behavior where the control points smoothly follow the target with natural overshoot and settling.

## Interaction Design

### Fixed Endpoints
- **P₀:** left endpoint (fixed at 100px from left edge)
- **P₃:** right endpoint (fixed at 100px from right edge)

### Dynamic Control Points
- **P₁ and P₂:**inner control points that respond to user input
- both points are attracted to a `globalTarget` with horizontal offsets to maintain curve shape
- **Offset calculation:** ±8% of the distance between endpoints (optimized for visual aesthetics)
- **Vertical offset:** P₂ offset by 15px for added visual interest

### Input Handling

1. **Mouse/Touch Movement:**
   - updates `globalTarget` position
   - P₁ and P₂ spring toward offset positions around the target
   - creates fluid rope-like motion

2. **Direct Control Point Dragging:**
   - click/touch within 20 pixels of P₁ or P₂ to grab them (increased hit area for better UX)
   - dragging directly sets position and zeros velocity
   - provides precise manual control
   - single-touch only for mobile (ignores multi-touch gestures)

3. **Idle Behavior:**
   - when no input is detected, points slowly drift toward center baseline
   - easing factor: 2% per frame for gentler, more natural drift

## Rendering Architecture

### Canvas Setup
- full viewport canvas with dark gradient background
- responsive resize handling with automatic fixed-point repositioning
- high DPI support with proper coordinate scaling for mouse/touch input
- canvas dimensions cached to avoid repeated DOM queries

### Render Order (back to front):
1. **Control Polygon:** dashed white lines connecting all control points (8% opacity)
2. **Bézier Curve:** bright cyan line (`#55c7ff`), 4px width with glow effect, sampled at 0.01 intervals (101 points)
3. **Tangent Lines:**
   - yellow indicators every 0.1 along curve (11 tangent lines)
   - Zzero-length tangent protection (checks for tanLen > 0.001)
   - 28px line length centered at sample points
4. **Control Points:** 
   - P₀/P₃ (fixed): White fill with cyan stroke
   - P₁ (dynamic): Red tinted with red stroke
   - P₂ (dynamic): Green tinted with green stroke
5. **Velocity Arrows:** small directional indicators showing current velocity

### Performance Optimization
- **Target Frame Rate:** 60 FPS via `requestAnimationFrame`
- **Delta Time Clamping:** saximum 33.3ms (1/30s) per frame to prevent physics instability during frame drops
- **Efficient Rendering:** single-pass curve drawing with manual sampling
- **Memory Optimization:** reusable objects, no allocations in hot loops
- **Coordinate Caching:** canvas dimensions cached to avoid repeated layout queries

## Code Organization

The implementation is organized into logical sections:

1. **Utilities**: vector math operations (add, subtract, scale, normalize, length)
2. **Canvas Setup**: responsive canvas with resize handling and coordinate caching
3. **Bézier Math**: oure mathematical functions for cubic Bézier curve and first derivative
4. **Scene Management**: fixed endpoint updates and control point initialization
5. **Physics**: dynamicPoint class with spring-damper integration using semi-implicit Euler
6. **Input Handling**: mouse/touch event listeners with DPI-aware coordinate conversion
7. **Rendering**: canvas drawing with glow effects, tangent visualization, and safety checks
8. **Animation Loop**: fixed timestep physics loop with proper initialization sequence

## Design Choices

### Why Spring-Damping?
- creates natural, organic motion that feels responsive yet smooth
- prevents jarring instant movements
- simulates real-world rope physics without complex collision detection

### Why Manual Bézier Implementation?
- assignment requirement (no prebuilt APIs)
- full control over sampling rate and optimization
- educational value in understanding the mathematics

### Why Offset Targets?
- maintains aesthetic curve shape during interaction
- prevents control points from collapsing to the same position
- creates more interesting visual dynamics
- horizontal and vertical offsets produce natural S-curve behavior

### Color Coding
- different colors for fixed vs. dynamic points aid understanding
- tangent lines in contrasting yellow show mathematical derivative
- velocity arrows provide real-time physics feedback

## Technical Requirements Met

 **Manual Bézier Math:** complete implementation of B(t) formula  
 **Manual Tangent Calculation:** first derivative B'(t) implementation  
 **Spring-Damping Physics:** custom acceleration-based motion model  
 **Interactive Control:** mouse/touch input with dragging support  
 **Tangent Visualization:** normalized tangent lines at regular intervals  
 **60 FPS Performance:** optimized rendering with requestAnimationFrame  
 **Clean Code Organization:** separated math, physics, input, and rendering  
 **No External Libraries:** pure HTML5 Canvas + vanilla JavaScript  

## Running the Application

1. open `bezier_rope.html` in any modern web browser
2. move your mouse/finger across the canvas to influence the curve
3. click and drag the red (P₁) or green (P₂) control points for direct manipulation
4. observe the yellow tangent lines showing the curve's direction at each point

## Technical Improvements Made

### Accuracy Enhancements
 **DPI Scaling:** proper coordinate transformation for high-DPI displays  
 **Resize Handling:** fixed endpoints update dynamically on window resize  
 **Zero-Length Tangent Protection:** safety check prevents division by zero  
 **Optimized Coefficients:** pre-computed squared terms reduce redundant multiplications  
 **Single-Touch Handling:** prevents multi-touch gesture interference  

### Performance Improvements
 **Coordinate Caching:** canvas dimensions stored to avoid DOM queries  
 **Memory Optimization:** no object allocation in animation loop  
 **Tuned Physics:** optimal spring/damping parameters for responsive feel  
 **Precise Sampling:** 101 curve points and 11 tangent visualizations  

### Code Quality
 **Better Comments:** mathematical formulas documented in code  
 **Explicit Initialization:** clear initialization sequence at startup  
 **Improved Constants:** named constants for magic numbers  
 **Edge Case Safety:** null checks and bounds validation  

## Future Enhancements

- add multiple rope segments for chain-like behavior
- implement collision detection with canvas boundaries
- add gravity simulation for more realistic physics
- support for mobile device gyroscope input (iOS CoreMotion)
- adjustable physics parameters via UI controls
- cubic Hermite spline interpolation for smoother motion

## Submission

screen recording  showing interactivity.

link [https://drive.google.com/file/d/1YDPtT06Ryu4sYGt5weX82uXMb1p33f9k/view?usp=drive_link]