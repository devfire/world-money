# 2D Map Money Movement Visualization

This document explains the functionality and implementation details of the "2D Map Money Movement" visualization - an interactive web application that displays animated financial transactions between global cities on a world map.

## Overview

The visualization creates an animated world map showing simulated financial transactions between major cities. Each transaction is represented by a curved arrow that:

1. Displays the origin and destination city names
2. Animates along a curved path between the two locations
3. Shows the transaction amount that moves along the path
4. Uses color-coding to indicate transaction size (green for smaller amounts, yellow/orange/red for larger amounts)

## Technical Implementation

### Core Libraries

The visualization uses several JavaScript libraries:

- **Leaflet** (`leaflet.js`) - For base map functionality
- **Leaflet.curve** (`leaflet.curve.min.js`) - For drawing curved paths between locations
- **Anime.js** (`anime.min.js`) - For smooth animation handling
- **CARTO Basemaps** - For the light-colored map tiles

### HTML Structure

The HTML structure is minimal, consisting of:
- A full-viewport `map` div that contains the Leaflet map
- An `info` div positioned at the top center to display status information

### CSS Styling

Key CSS classes include:
- `.animated-arrow-path` - Styling for the transaction path
- `.transaction-label` - Styling for the moving transaction amount label
- `.city-label` - Styling for the temporary city name labels

### Core Functionality

#### Map Initialization

The map is initialized with a global view centered at coordinates [25, 0] with zoom level 2. It uses the CARTO light basemap for a clean visual appearance.

#### City Data

The visualization includes 13 major global cities defined with:
- City name
- Latitude
- Longitude

Each city is displayed as a small circle marker on the map with a white fill and dark border.

#### Transaction Animation

The core functionality is implemented in the `addTransactionArrow()` function, which:

1. **Creates a curved path** between origin and destination:
   ```javascript
   const pathData = ['M', [latLng1.lat, latLng1.lng], 'Q', [controlLatLng.lat, controlLatLng.lng], [latLng2.lat, latLng2.lng]];
   ```

2. **Determines visual properties** based on transaction amount:
   - Larger transactions have thicker lines and redder colors
   - Smaller transactions have thinner lines and greener colors
   - The color is determined by HSL with hue values from 0-120

3. **Adds multiple elements** to the map:
   - The curved arrow path
   - A transaction amount label that follows the path
   - Origin city name label
   - Destination city name label

4. **Creates an animation timeline** using Anime.js with several phases:
   - Fade in all elements
   - Draw the arrow while moving the amount label
   - Hold the completed state briefly
   - Fade out all elements
   - Remove all elements from the map

5. **Handles errors** through try/catch to ensure cleanup even if animation fails

#### Continuous Simulation

The visualization continuously generates random transactions through the `simulateTransaction()` function, which:

1. Selects two different random cities
2. Generates a random transaction amount
3. Creates and animates a transaction between the cities
4. Sets a random timeout (500-2000ms) before starting the next transaction

### Helper Functions

- `formatCurrency()` - Formats numerical values as USD currency strings
- `getRandomAmount()` - Generates random transaction amounts between $1,000 and $10,000,000

## Technical Details

### Arrow Path Calculation

The curved path between cities is calculated as a quadratic Bézier curve:
- The start and end points are the city coordinates
- The control point is positioned above both cities to create an arc
- For paths crossing the international date line, special handling adjusts the longitude values

```javascript
const latOffset = Math.abs(latLng1.lat - latLng2.lat) * 0.6 + 5;
const controlLat = Math.max(latLng1.lat, latLng2.lat) + latOffset;
let midLon = (latLng1.lng + latLng2.lng) / 2;
// Special handling for date line crossing
const lonDiff = Math.abs(latLng1.lng - latLng2.lng);
if (lonDiff > 180) { 
    midLon += 180; 
    if (midLon > 180) midLon -= 360; 
    if (midLon < -180) midLon += 360; 
}
```

### Animation Technique

The animation uses several advanced techniques:

1. **Path drawing animation** using SVG's `strokeDasharray` and `strokeDashoffset` properties
2. **Label movement animation** using a helper object to update the marker's position on each frame
3. **Timeline sequencing** to coordinate multiple animations
4. **Cleanup handling** to remove all elements after animation completes

### Error Handling

The code includes error handling to prevent issues from breaking the visualization:
- Try/catch blocks around animation creation
- Element existence checks
- Error logging to console
- Cleanup of all elements even when errors occur

## Performance Considerations

The visualization is optimized in several ways:

1. **Limited DOM elements** - Only a few transactions are active at once
2. **Cleanup after animations** - All elements are removed after their animation completes
3. **Minimal reflows** - Animation uses efficient properties like opacity and stroke properties
4. **Pointer-events disabled** - Moving labels don't interfere with map interaction

## Browser Compatibility

The visualization requires browsers that support:
- Modern JavaScript (ES6+)
- SVG animations
- CSS transforms
- The Intl.NumberFormat API for currency formatting

## Enhancement Possibilities

The visualization could be extended with:
- Real-time transaction data from an API
- Additional transaction details on hover/click
- Filtering options for transaction types or amounts
- Time-based playback controls
- Clustering for high-volume city pairs
