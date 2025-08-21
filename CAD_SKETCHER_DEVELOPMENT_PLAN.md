# CAD Sketcher Integration Development Plan for Blender MCP

## Overview
This document outlines the comprehensive plan to expand the Blender MCP client with full CAD Sketcher addon control capabilities.

## Project Scope

### Primary Goals
- Complete 2D sketching capabilities with all entity types
- Full constraint system support
- Parametric dimensions and expressions
- 3D operations (extrude, revolve, loft, sweep)
- Modal and non-modal execution parity
- Robust error handling and status reporting

### Acceptance Criteria

#### 2D Entities
- Points, lines, circles, arcs (all construction methods)
- Rectangles, polygons, slots
- Fillets, chamfers, offsets
- Polylines and splines
- Construction geometry flag support

#### Constraints
- Geometric: coincident, colinear, parallel, perpendicular, tangent
- Equality: equal length/radius/angle
- Dimensional: distance, radius, angle
- Position: horizontal, vertical, midpoint, point-on-line
- Advanced: concentric, symmetric, fix/lock

#### Dimensions & Parameters
- Create, bind, rename dimensions
- Expression evaluation engine
- Unit conversion support (m, cm, mm, in)
- Global and sketch-scoped parameters
- Update propagation with dependencies

#### Sketch Operations
- Transform: move, rotate, scale, mirror
- Edit: offset, trim, extend, delete
- Fillet/chamfer with parametric control
- Grouping and tag assignment

#### 3D Operations
- Extrude: join/new/cut/intersect modes
- Revolve around axis
- Loft/sweep between profiles
- Thickness operations
- Profile selection from sketches
- Parameter-driven operations

## Architecture

### Server-Side Components (Python)

```
mcp_blender/
├── cad_sketcher/
│   ├── __init__.py
│   ├── session.py       # Sketch session management
│   ├── entities.py      # Entity CRUD operations
│   ├── constraints.py   # Constraint management
│   ├── dimensions.py    # Dimensions and parameters
│   ├── transform.py     # Transformation operations
│   ├── ops3d.py        # 3D operations
│   └── utils.py        # Utilities and helpers
├── runtime.py          # Main-thread execution
└── errors.py           # Error taxonomy
```

### MCP Tool Structure

#### Sketch Management
- `cad/sketch/create` - Create new sketch
- `cad/sketch/activate` - Activate sketch for editing
- `cad/sketch/close` - Close active sketch
- `cad/sketch/delete` - Delete sketch
- `cad/sketch/list` - List all sketches

#### Entity Operations
- `cad/entity/add` - Add entity with type and method
- `cad/entity/update` - Update entity properties
- `cad/entity/delete` - Delete entity
- `cad/entity/get` - Get entity details
- `cad/entity/list` - List entities in sketch

#### Constraint Operations
- `cad/constraint/add.<type>` - Add specific constraint
- `cad/constraint/delete` - Remove constraint
- `cad/constraint/list` - List constraints
- `cad/constraint/get` - Get constraint details

#### Dimension Operations
- `cad/dimension/add.<type>` - Add dimension
- `cad/param/set_value` - Set parameter value
- `cad/param/rename` - Rename parameter
- `cad/param/list` - List parameters

#### Transformation Operations
- `cad/sketch/transform.<op>` - Apply transformation
- `cad/solver/solve` - Run constraint solver
- `cad/analysis/dof` - Get degrees of freedom
- `cad/analysis/bbox` - Get bounding box

#### 3D Operations
- `cad3d/extrude` - Extrude profile
- `cad3d/revolve` - Revolve profile
- `cad3d/loft` - Loft between profiles
- `cad3d/sweep` - Sweep along path

## Data Model

### ID Management
- UUID v4 for all entities, constraints, dimensions
- Mapping to CAD Sketcher internal IDs
- Sub-element references: `entity_id@v0` (vertex), `entity_id@e0` (edge)

### Coordinate Systems
- World space: meters, right-handed
- Sketch plane: local 2D (u, v)
- Plane definition: `{origin, x_axis, normal}`

### Parameters
- Scoped: global or per-sketch
- Expression support with dependency resolution
- Unit-qualified values ("25 mm")
- Default session unit configurable

## Implementation Phases

### Phase 1: Foundation (Current)
✅ Basic point/line operations
✅ Simple solver calls
✅ MCP tool wrappers
⬜ Capability detection
⬜ Error handling improvements

### Phase 2: Core 2D Features
⬜ All entity types
⬜ Complete constraint system
⬜ Dimension creation
⬜ Parameter management
⬜ Expression evaluation

### Phase 3: Advanced 2D
⬜ Transformations
⬜ Editing operations
⬜ Construction geometry
⬜ Profile loops
⬜ Selection management

### Phase 4: 3D Operations
⬜ Extrude implementation
⬜ Revolve implementation
⬜ Loft/sweep
⬜ Boolean operations
⬜ Parametric modifiers

### Phase 5: Polish & Testing
⬜ Comprehensive tests
⬜ Documentation
⬜ Performance optimization
⬜ CI/CD setup
⬜ Example workflows

## Technical Decisions

### Threading Model
- All Blender/addon operations on main thread
- Queue via `bpy.app.timers.register`
- Reject concurrent modal operations
- Serialize transactions

### Error Handling
- Typed error taxonomy
- Structured error payloads
- Graceful capability degradation
- Detailed conflict reporting

### Performance
- Batch constraint operations
- Debounce parameter updates
- Cache ID mappings
- Minimize solver invocations

### Testing Strategy
- Unit tests with pytest-bpy
- Integration tests via MCP
- Cross-platform validation
- CI matrix: OS × Blender versions

## Migration & Compatibility

### Backward Compatibility
- Preserve existing tool names
- Add new tools under `cad/*` namespace
- Feature flags for advanced operations
- Graceful degradation

### Version Support
- Blender 3.0+
- CAD Sketcher 0.27+
- Python 3.10+
- MCP protocol 1.3.0+

## Next Steps

1. **Immediate** (Today):
   - Implement capability detection
   - Improve error handling for existing tools
   - Add cad/sketch/list tool

2. **This Week**:
   - Complete circle/arc entity types
   - Add basic constraints (parallel, perpendicular)
   - Implement dimension creation

3. **Next Week**:
   - Parameter management system
   - Expression evaluation
   - Transformation operations

## Notes for Future Sessions

This plan was created on 2025-08-21 for Don Tadeo. The CAD Sketcher integration is partially implemented with basic point/line operations working. The next priority is expanding entity type coverage and implementing the constraint system properly.

Key files to review:
- `/addon.py` - Lines 235-683 contain current CAD implementation
- `/src/blender_mcp/server.py` - Lines 940-1022 contain MCP tool wrappers

Remember: Always use Context7 MCP patterns, maintain backward compatibility, and ensure all operations run on Blender's main thread.
