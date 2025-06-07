# Button Avoidance Script Documentation

## Overview

The Button Avoidance System is a JavaScript class that creates an interactive UI element that dynamically moves away from the user's mouse cursor, creating a playful and engaging user experience. This system is commonly used for creating "hard to click" buttons for humorous or attention-grabbing purposes.

## Core Architecture

### ButtonAvoidance Class

The system is built around a single ES6 class `ButtonAvoidance` that encapsulates all functionality related to mouse tracking, distance calculation, and button movement.

```javascript
class ButtonAvoidance {
    constructor(button, avoidanceRadius = 100, moveSpeed = 0.3)
}
```

## Constructor Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `button` | HTMLElement | required | The DOM element to apply avoidance behavior to |
| `avoidanceRadius` | Number | 100 | Distance in pixels at which the button starts avoiding the mouse |
| `moveSpeed` | Number | 0.3 | Movement speed multiplier (0.1 = slow, 1.0 = instant) |

## Instance Properties

### Position Tracking
- `buttonX`: Current X coordinate of the button center
- `buttonY`: Current Y coordinate of the button center
- `mouseX`: Current mouse X coordinate
- `mouseY`: Current mouse Y coordinate

### Behavior Configuration
- `avoidanceRadius`: Detection radius for mouse proximity
- `moveSpeed`: Movement easing factor
- `isAvoiding`: Boolean flag indicating if button is currently avoiding mouse

## Core Methods

### `init()`
**Purpose**: Initializes the avoidance system and sets up event listeners.

**Functionality**:
- Sets initial button position to center of viewport
- Attaches `mousemove` event listener for real-time mouse tracking
- Attaches `resize` event listener for window boundary management
- Sets up button click handler for navigation

```javascript
init() {
    this.updateButtonPosition();
    document.addEventListener('mousemove', (e) => {
        this.mouseX = e.clientX;
        this.mouseY = e.clientY;
        this.checkAvoidance();
    });
    // ... additional event listeners
}
```

### `checkAvoidance()`
**Purpose**: Determines if the mouse is close enough to trigger avoidance behavior.

**Algorithm**:
1. Calculates distance between mouse and button using `getDistance()`
2. If distance < `avoidanceRadius`, calls `avoidMouse()`
3. Provides the trigger condition for avoidance activation

### `avoidMouse()`
**Purpose**: Core avoidance logic that calculates and applies button movement.

**Mathematical Approach**:
1. **Vector Calculation**: Computes direction vector from mouse to button
   ```javascript
   const deltaX = this.buttonX - this.mouseX;
   const deltaY = this.buttonY - this.mouseY;
   ```

2. **Normalization**: Converts direction to unit vector
   ```javascript
   const distance = Math.sqrt(deltaX * deltaX + deltaY * deltaY);
   const moveX = (deltaX / distance) * (this.avoidanceRadius - distance + 50);
   ```

3. **Movement Application**: Applies eased movement in calculated direction
   ```javascript
   this.buttonX += moveX * this.moveSpeed;
   this.buttonY += moveY * this.moveSpeed;
   ```

4. **Randomization**: Adds occasional random movement for unpredictability
   ```javascript
   if (Math.random() > 0.8) {
       this.buttonX += (Math.random() - 0.5) * 100;
       this.buttonY += (Math.random() - 0.5) * 100;
   }
   ```

### `constrainToWindow()`
**Purpose**: Ensures the button remains within visible viewport boundaries.

**Implementation**:
- Calculates button dimensions using `getBoundingClientRect()`
- Applies margin buffer to prevent edge clipping
- Clamps position values to valid ranges

```javascript
constrainToWindow() {
    const buttonRect = this.button.getBoundingClientRect();
    const margin = 50;
    
    this.buttonX = Math.max(margin, Math.min(window.innerWidth - buttonRect.width - margin, this.buttonX));
    this.buttonY = Math.max(margin, Math.min(window.innerHeight - buttonRect.height - margin, this.buttonY));
}
```

### `updateButtonPosition()`
**Purpose**: Applies calculated position to the DOM element.

**CSS Application**:
- Sets `position: fixed` on button element
- Updates `left` and `top` CSS properties
- Triggers browser repaint for visual update

### `getDistance(x1, y1, x2, y2)`
**Purpose**: Utility function for Euclidean distance calculation.

**Formula**: 
```
distance = √[(x₂-x₁)² + (y₂-y₁)²]
```

## Performance Considerations

### Event Throttling
The system uses direct `mousemove` event binding without throttling. For performance optimization in complex applications, consider implementing:

```javascript
// Throttled version example
let lastUpdate = 0;
document.addEventListener('mousemove', (e) => {
    const now = Date.now();
    if (now - lastUpdate > 16) { // ~60fps
        this.mouseX = e.clientX;
        this.mouseY = e.clientY;
        this.checkAvoidance();
        lastUpdate = now;
    }
});
```

### CSS Transitions
The button uses CSS transitions for smooth movement:
```css
.avoid-button {
    transition: all 0.3s cubic-bezier(0.25, 0.46, 0.45, 0.94);
}
```

## Usage Example

```javascript
// Initialize avoidance system
const button = document.getElementById('avoidButton');
const avoidanceSystem = new ButtonAvoidance(button, 120, 0.4);

// System automatically activates once instantiated
// No additional method calls required
```

## Configuration Options

### Sensitivity Tuning
- **High Sensitivity**: `avoidanceRadius: 150, moveSpeed: 0.6`
- **Low Sensitivity**: `avoidanceRadius: 80, moveSpeed: 0.2`
- **Aggressive Avoidance**: `avoidanceRadius: 200, moveSpeed: 0.8`

### Behavior Modifications
- **Predictable Movement**: Remove randomization block in `avoidMouse()`
- **Boundary Escape**: Increase margin in `constrainToWindow()`
- **Sticky Edges**: Add edge-detection logic for corner trapping

## Browser Compatibility

- **Modern Browsers**: Full support (Chrome 60+, Firefox 55+, Safari 12+)
- **Event Support**: `mousemove`, `resize` events universally supported
- **CSS Requirements**: `position: fixed`, CSS transitions
- **JavaScript Features**: ES6 classes, arrow functions, `getBoundingClientRect()`

## Debugging Tools

### Console Logging
Add debug output to track system behavior:
```javascript
checkAvoidance() {
    const distance = this.getDistance(this.mouseX, this.mouseY, this.buttonX, this.buttonY);
    console.log(`Distance: ${distance}, Avoiding: ${distance < this.avoidanceRadius}`);
    // ... rest of method
}
```

### Visual Debugging
Add visual indicators for avoidance radius:
```javascript
// Add circle overlay to visualize detection radius
const debugCircle = document.createElement('div');
debugCircle.style.cssText = `
    position: fixed;
    border: 2px dashed red;
    border-radius: 50%;
    width: ${this.avoidanceRadius * 2}px;
    height: ${this.avoidanceRadius * 2}px;
    pointer-events: none;
`;
```

## Common Issues & Solutions

### Button Stuck in Corner
**Problem**: Button gets trapped in viewport corners
**Solution**: Increase margin in `constrainToWindow()` or add corner-escape logic

### Jittery Movement
**Problem**: Button appears to shake or vibrate
**Solution**: Reduce `moveSpeed` or implement movement smoothing

### Poor Performance
**Problem**: High CPU usage during mouse movement
**Solution**: Implement event throttling or use `requestAnimationFrame`

### Button Not Responding
**Problem**: Button doesn't move away from mouse
**Solution**: Verify button has `position: fixed` CSS and check console for errors