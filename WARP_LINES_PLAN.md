# Warp Lines Game - Complete Implementation Plan

## Project Overview
**Warp Lines** is a kid-friendly, abstract strategy board game inspired by checkers/draughts but played on a custom graph-based board. This document outlines the complete implementation plan for creating a single-file HTML5 version of the game.

---

## Game Summary

### Core Concept
- **Players**: 2 (Red vs Green)
- **Pieces**: 13 per player
- **Board**: 27-node graph (9 rows × 3 columns with specific connectivity)
- **Goal**: Eliminate opponent's pieces or block all their moves
- **Style**: Turn-based abstract strategy with mandatory capture rules

### Target Audience
- Children aged 7-12 years
- Emphasis on strategic thinking, spatial reasoning, and turn-taking
- Simple rules, clear visuals, positive feedback

---

## Board Topology (FIXED - DO NOT MODIFY)

### Node Layout
```
Row 1 (top):    A1  A2  A3
Row 2:          B1  B2  B3
Row 3:          C1  C2  C3
Row 4:          D1  D2  D3
Row 5 (center): E1  E2  E3
Row 6:          F1  F2  F3
Row 7:          G1  G2  G3
Row 8:          H1  H2  H3
Row 9 (bottom): I1  I2  I3
```

### Edge Connections (COMPLETE LIST)
1. **Horizontal edges** (every row):
   - A1–A2–A3, B1–B2–B3, C1–C2–C3, D1–D2–D3, E1–E2–E3
   - F1–F2–F3, G1–G2–G3, H1–H2–H3, I1–I2–I3

2. **Central vertical spine**:
   - A2–B2–C2–D2–E2–F2–G2–H2–I2

3. **Upper-left inner diagonal**:
   - A1–B1–C1–D1–E2

4. **Upper-right inner diagonal**:
   - A3–B3–C3–D3–E2

5. **Lower-left inner diagonal**:
   - E2–F1–G1–H1–I1

6. **Lower-right inner diagonal**:
   - E2–F3–G3–H3–I3

7. **Left outer border**:
   - A1–E1–I1

8. **Right outer border**:
   - A3–E3–I3

### Starting Position
- **Red pieces** (13): A1, A2, A3, B1, B2, B3, C1, C2, C3, D1, D2, D3, E1
- **Green pieces** (13): F1, F2, F3, G1, G2, G3, H1, H2, H3, I1, I2, I3, E3
- **Empty**: E2 (center hub)

---

## Game Rules

### Turn Structure
1. Players alternate turns (Red starts)
2. On each turn, player MUST make exactly ONE move
3. A move is either:
   - **Normal Step**: Move to adjacent empty node
   - **Capture**: Jump over enemy piece(s)

### Mandatory Capture Rule
- If ANY of your pieces can capture, you MUST capture
- You cannot make a normal step when a capture is available
- The game engine must check all pieces before allowing moves

### Normal Step Rules
- Select your piece on node X
- Move to adjacent empty node Y (where neighbors[X] contains Y)
- Cannot move more than one edge
- Cannot move to occupied node

### Capture (Jump) Rules
#### Single Jump
- Your piece on node A
- Enemy piece on adjacent node B
- Empty node C where B is adjacent to C (A–B–C along edges)
- Jump from A to C, removing enemy piece on B

#### Multi-Jump
- After landing, if another jump is possible, you MUST continue
- Can change direction between jumps
- Continues until no more jumps are possible
- Each jump segment must follow A–B–C pattern

### Prohibited Actions
- Jumping over your own pieces
- Jumping over multiple pieces in one segment
- Landing on occupied nodes
- Stopping mid-multi-jump when captures remain

### Win Conditions
1. **No moves**: Opponent has no legal moves at turn start
2. **Elimination**: Opponent has zero pieces on board

### Optional (Future)
- Draw by repetition (same position 3 times)
- Draw by no captures in 30 turns

---

## Implementation Phases

### Phase 1: Core Game Engine
**Files**: Foundation of game logic

#### Data Structures
```javascript
// Node adjacency list
const neighbors = {
  "A1": ["A2", "B1", "E1"],
  "A2": ["A1", "A3", "B2"],
  // ... complete mapping for all 27 nodes
};

// Game state
const gameState = {
  board: {}, // nodeId -> "red" | "green" | null
  currentPlayer: "red",
  selectedPiece: null,
  mode: "pvp", // "pvp" or "pvc"
  gameOver: false,
  winner: null,
  moveHistory: []
};
```

#### Core Functions
- `initGame()`: Initialize game state
- `resetGame()`: Reset to starting position
- `isValidNode(nodeId)`: Check if node exists
- `getPieceAt(nodeId)`: Get piece color at node
- `setPieceAt(nodeId, color)`: Set piece at node
- `getNeighbors(nodeId)`: Get adjacent nodes

### Phase 2: Move Generation & Validation
**Files**: Move logic and validation

#### Functions
- `getLegalMoves(player)`: Get all legal moves for player
- `getLegalMovesForPiece(nodeId)`: Get moves for specific piece
- `getStepMoves(nodeId)`: Get normal step options
- `getCaptureMovesForPiece(nodeId)`: Get capture sequences
- `findCaptureSequence(start, current, captured, path)`: Recursive multi-jump finder
- `hasCaptureAvailable(player)`: Check if any capture exists
- `isValidMove(from, to, player)`: Validate specific move
- `applyMove(from, to, capturedNodes)`: Execute move

### Phase 3: Board Rendering
**Files**: Visual representation

#### Coordinate System
```javascript
// Map nodes to canvas coordinates
const nodePositions = {
  "A1": {x: 100, y: 50},
  "A2": {x: 300, y: 50},
  // ... coordinates for visual layout
};
```

#### Rendering Functions
- `drawBoard()`: Draw all edges and nodes
- `drawEdge(node1, node2)`: Draw single edge line
- `drawNode(nodeId)`: Draw node circle
- `drawPiece(nodeId, color)`: Draw piece at node
- `highlightNode(nodeId, type)`: Highlight for selection/target
- `animateMove(from, to, callback)`: Animate piece movement
- `animateCapture(nodeId)`: Animate piece removal

#### Visual Style
- **Background**: Dark slate (#2c3e50)
- **Edges**: Thin white lines (#ecf0f1)
- **Nodes**: Small circles, semi-transparent
- **Pieces**:
  - Red: #e74c3c (bright red)
  - Green: #2ecc71 (bright green)
  - Gradient effect for 3D marble look
- **Highlights**:
  - Selected: Yellow glow
  - Valid targets: Green glow (steps) or orange glow (captures)

### Phase 4: User Interaction
**Files**: Input handling

#### Click/Touch Handler
```javascript
function handleClick(canvasX, canvasY) {
  const nodeId = getNodeAtPosition(canvasX, canvasY);
  if (!nodeId) return;

  if (selectedPiece === null) {
    // Select piece
    if (getPieceAt(nodeId) === currentPlayer) {
      selectPiece(nodeId);
    }
  } else {
    // Execute move or deselect
    if (isValidTarget(nodeId)) {
      executeMove(selectedPiece, nodeId);
    } else {
      deselectPiece();
    }
  }
}
```

#### Functions
- `getNodeAtPosition(x, y)`: Hit detection for nodes
- `selectPiece(nodeId)`: Highlight piece and show valid moves
- `deselectPiece()`: Clear selection
- `showValidMoves(nodeId)`: Highlight legal target nodes
- `executeMove(from, to)`: Process complete move including captures
- `switchTurn()`: Change active player

### Phase 5: AI Implementation
**Files**: Computer opponent logic

#### AI Strategy
```javascript
function aiMove() {
  const allMoves = getLegalMoves("green");

  // Scoring criteria:
  // 1. Prefer captures (count captured pieces)
  // 2. Prefer moves leading to more mobility
  // 3. Prefer central positions (proximity to E2)

  const scoredMoves = allMoves.map(move => ({
    move,
    score: evaluateMove(move)
  }));

  const bestMove = scoredMoves.sort((a, b) => b.score - a.score)[0];
  return bestMove.move;
}
```

#### Functions
- `aiMove()`: Select and execute AI move
- `evaluateMove(move)`: Score move quality
- `countCapturedPieces(move)`: Count captures in move
- `calculateMobility(boardState, player)`: Count legal moves after move
- `getDistanceToCenter(nodeId)`: Distance to E2 (for positional scoring)

### Phase 6: Game Flow & UI
**Files**: Complete game management

#### Game Loop
- Check for game over conditions at turn start
- Display current player
- Handle player input or trigger AI
- Validate and execute moves
- Switch turns
- Update display

#### UI Components
- **Status bar**: Current player, message display
- **Game board**: Canvas rendering
- **Control buttons**:
  - "New Game"
  - "Mode: PvP / PvC" toggle
  - "Show Rules"
  - "Sound On/Off" (optional)
- **Rules overlay**: Modal with game rules
- **Win screen**: Display winner with play again option

#### Functions
- `updateStatus(message)`: Update status text
- `checkWinCondition()`: Check for game end
- `displayWinner(player)`: Show win screen
- `showRulesModal()`: Display rules overlay
- `toggleMode()`: Switch between PvP and PvC
- `handleNewGame()`: Reset and start new game

---

## Technical Implementation Details

### HTML Structure
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Warp Lines</title>
  <style>
    /* All CSS inline */
  </style>
</head>
<body>
  <div id="game-container">
    <div id="status-bar"></div>
    <canvas id="game-canvas"></canvas>
    <div id="controls"></div>
  </div>
  <div id="rules-modal" class="hidden"></div>
  <div id="win-modal" class="hidden"></div>

  <script>
    /* All JavaScript inline */
  </script>
</body>
</html>
```

### Key Technical Considerations

#### Responsive Design
- Canvas scales to viewport
- Touch events for mobile
- Minimum size constraints for small screens

#### Performance
- Efficient rendering (only redraw on state change)
- Request animation frame for animations
- Minimal DOM manipulation

#### Accessibility
- High contrast colors
- Large touch targets
- Clear feedback messages
- Keyboard support (future enhancement)

#### Browser Compatibility
- Pure HTML5 Canvas API
- No external dependencies
- ES6 JavaScript (widely supported)

---

## Testing Checklist

### Core Mechanics
- [ ] Board topology matches specification exactly
- [ ] Starting position correct (13 pieces each, E2 empty)
- [ ] Pieces move only along valid edges
- [ ] Cannot move to occupied nodes
- [ ] Cannot move to non-adjacent nodes

### Capture Rules
- [ ] Single jumps work correctly (A–B–C pattern)
- [ ] Multi-jumps continue until no captures remain
- [ ] Captured pieces are removed immediately
- [ ] Cannot jump over own pieces
- [ ] Cannot jump over multiple pieces in one segment
- [ ] Cannot land on occupied nodes

### Mandatory Capture
- [ ] Game detects when captures are available
- [ ] Normal steps blocked when capture exists
- [ ] All pieces checked for captures
- [ ] UI shows only capture options when required

### Win Conditions
- [ ] Win by elimination detected
- [ ] Win by no moves detected
- [ ] Game prevents moves after game over
- [ ] Winner displayed correctly

### AI Behavior
- [ ] AI obeys all rules
- [ ] AI makes captures when required
- [ ] AI completes multi-jumps
- [ ] AI doesn't hang or error
- [ ] AI moves feel reasonable

### UI/UX
- [ ] Pieces selectable by current player only
- [ ] Valid moves highlighted correctly
- [ ] Move animations smooth
- [ ] Status messages clear
- [ ] Controls work on desktop
- [ ] Controls work on mobile touch
- [ ] New game resets properly
- [ ] Mode toggle works

---

## Educational Benefits

### Cognitive Skills Developed
1. **Strategic Thinking**: Planning moves ahead, anticipating opponent
2. **Spatial Reasoning**: Understanding graph connectivity vs grid
3. **Pattern Recognition**: Identifying capture opportunities
4. **Problem Solving**: Finding optimal paths and sequences
5. **Decision Making**: Weighing trade-offs between moves

### Social Skills
1. **Turn-Taking**: Patience and waiting
2. **Fair Play**: Following rules consistently
3. **Handling Wins/Losses**: Emotional regulation
4. **Social Interaction**: Communication in pass-and-play

### Mathematical Concepts
1. **Graph Theory**: Nodes, edges, connectivity
2. **Combinatorics**: Counting possible moves
3. **Logic**: If-then reasoning for rules
4. **Geometry**: Spatial relationships

---

## Future Enhancements (Post-MVP)

### Phase 7: Polish & Features
- [ ] Sound effects (move, capture, win)
- [ ] Background music toggle
- [ ] Animation speed control
- [ ] Undo move (in PvP mode)
- [ ] Move history display
- [ ] Save/load game state (localStorage)

### Phase 8: Advanced Features
- [ ] Multiple AI difficulty levels (easy, medium, hard)
- [ ] Hint system (show suggested move)
- [ ] Tutorial mode (guided first game)
- [ ] Statistics tracking (wins, captures, etc.)
- [ ] Achievements system

### Phase 9: Online Multiplayer (Separate Project)
- [ ] Firebase integration
- [ ] Turn-based matchmaking
- [ ] Room codes for private games
- [ ] Game state synchronization
- [ ] Player profiles
- [ ] Leaderboards

### Phase 10: Mobile App (Native)
- [ ] Unity implementation
- [ ] Enhanced 3D graphics
- [ ] More sophisticated AI
- [ ] Play Games Services integration
- [ ] In-app purchases (themes, etc.)

---

## File Structure (Single HTML File)

```
warp-lines.html
├── <!DOCTYPE html>
├── <head>
│   ├── <meta> tags
│   ├── <title>
│   └── <style> (all CSS inline)
│       ├── Reset styles
│       ├── Layout styles
│       ├── Game board styles
│       ├── Control styles
│       ├── Modal styles
│       └── Responsive styles
├── <body>
│   ├── Game container
│   ├── Canvas element
│   ├── Control buttons
│   ├── Modals (rules, win)
│   └── <script> (all JavaScript inline)
│       ├── Constants & Configuration
│       ├── Data Structures
│       ├── Game State
│       ├── Core Functions
│       ├── Move Generation
│       ├── Move Validation
│       ├── Move Execution
│       ├── AI Functions
│       ├── Rendering Functions
│       ├── Input Handlers
│       ├── UI Functions
│       └── Initialization
└── </html>
```

---

## Code Organization

### Section 1: Configuration
- Canvas dimensions
- Colors and styles
- Node positions
- Neighbor adjacency list

### Section 2: Game State
- Board state object
- Current player tracking
- Selection state
- Mode settings

### Section 3: Utility Functions
- Node validation
- Piece getting/setting
- Neighbor queries
- Distance calculations

### Section 4: Move Logic
- Step move generation
- Capture detection
- Multi-jump recursion
- Move validation
- Move execution

### Section 5: AI Logic
- Move evaluation
- Position scoring
- Decision making

### Section 6: Rendering
- Canvas setup
- Board drawing
- Piece drawing
- Animation functions

### Section 7: Input Handling
- Click detection
- Touch handling
- Piece selection
- Move execution

### Section 8: UI Management
- Status updates
- Button handlers
- Modal management
- Game flow control

### Section 9: Initialization
- DOM ready handler
- Initial game setup
- Event listener binding

---

## Common Pitfalls to Avoid

### ❌ Graph Topology Errors
- Adding extra edges not in specification
- Treating board as regular grid
- Allowing diagonal moves where not connected

### ❌ Capture Rule Violations
- Forgetting mandatory capture check
- Allowing early stop in multi-jump
- Not removing captured pieces immediately
- Wrong A–B–C validation

### ❌ State Management Issues
- Not updating board after move
- Keeping stale selection state
- Not switching turns properly
- Not checking win conditions

### ❌ UI/UX Problems
- Small touch targets on mobile
- Unclear which moves are legal
- No feedback on illegal moves
- Confusing turn indicators

### ❌ AI Bugs
- Making illegal moves
- Violating mandatory capture
- Infinite loops in move search
- Not handling no-move situations

---

## Performance Targets

- **Initial Load**: < 1 second
- **Move Response**: < 100ms (instant feel)
- **AI Move Time**: < 2 seconds (easy), < 5 seconds (hard)
- **Animation Duration**: 300-500ms per segment
- **Frame Rate**: Smooth 60fps during animations
- **File Size**: < 50KB (single HTML file)

---

## Browser Support

### Minimum Requirements
- **Desktop**: Chrome 90+, Firefox 88+, Safari 14+, Edge 90+
- **Mobile**: iOS Safari 14+, Chrome Android 90+
- **Canvas API**: Full support required
- **ES6**: Arrow functions, const/let, template literals

---

## Accessibility Considerations

### Current Implementation
- High contrast colors (WCAG AA compliant)
- Large touch targets (min 44x44px)
- Clear status messages
- Simple, readable fonts

### Future Improvements
- Keyboard navigation
- Screen reader support (ARIA labels)
- Alternative color schemes for colorblind users
- Adjustable text size
- Reduced motion option

---

## Development Workflow

### Step 1: Create Structure
1. Set up HTML skeleton
2. Add CSS for layout
3. Create canvas element
4. Test basic rendering

### Step 2: Implement Core
1. Define adjacency list
2. Create game state
3. Implement move generation
4. Add move validation
5. Unit test move logic

### Step 3: Add Visuals
1. Draw board edges
2. Draw nodes
3. Draw pieces
4. Test on different screen sizes

### Step 4: Add Interaction
1. Click detection
2. Piece selection
3. Move highlights
4. Move execution
5. Turn switching

### Step 5: Implement AI
1. Basic move selection
2. Capture preference
3. Position evaluation
4. Test against human

### Step 6: Polish
1. Add animations
2. Add status messages
3. Add win detection
4. Add controls
5. Add rules screen

### Step 7: Test & Refine
1. Test all rules
2. Test edge cases
3. Test on mobile
4. Fix bugs
5. Optimize performance

---

## Success Criteria

### Functional Requirements
✅ Game follows all specified rules exactly
✅ Both PvP and PvC modes work
✅ AI makes legal moves only
✅ Win conditions detected correctly
✅ Runs in single HTML file
✅ Works on desktop and mobile

### Quality Requirements
✅ Code is well-commented
✅ No console errors
✅ Smooth animations
✅ Responsive design
✅ Clear UI feedback
✅ Kid-friendly aesthetic

### Performance Requirements
✅ Loads quickly
✅ Responds instantly to input
✅ AI moves in reasonable time
✅ No lag during gameplay

---

## Project Timeline

### Phase 1-2: Foundation (Day 1)
- Data structures
- Move logic
- Validation

### Phase 3-4: Visuals (Day 1-2)
- Rendering
- Interaction

### Phase 5-6: Completion (Day 2-3)
- AI
- Polish
- Testing

**Total Estimated Time**: 2-3 days for single developer

---

## Resources & References

### Game Design
- Reference image provided
- Graph-based board games (Hex, Y, etc.)
- Checkers/Draughts rules for mandatory capture

### Technical
- HTML5 Canvas API documentation
- Touch event handling
- Minimax algorithm for AI
- Graph traversal algorithms

### Similar Games
- Checkers/Draughts
- Chinese Checkers
- Halma
- Konane

---

## Contact & Support

For questions, issues, or suggestions about Warp Lines:
- Review this document thoroughly before implementation
- Test each phase completely before moving to next
- Keep code clean and well-commented
- Prioritize correctness over complexity

---

## Version History

- **v1.0 (Current)**: Initial planning document
  - Complete rule specification
  - Implementation phases outlined
  - Testing checklist defined

---

## Appendix A: Complete Adjacency List

```javascript
const neighbors = {
  // Row 1
  "A1": ["A2", "B1", "E1"],
  "A2": ["A1", "A3", "B2"],
  "A3": ["A2", "B3", "E3"],

  // Row 2
  "B1": ["A1", "B2", "C1"],
  "B2": ["A2", "B1", "B3", "C2"],
  "B3": ["A3", "B2", "C3"],

  // Row 3
  "C1": ["B1", "C2", "D1"],
  "C2": ["B2", "C1", "C3", "D2"],
  "C3": ["B3", "C2", "D3"],

  // Row 4
  "D1": ["C1", "D2", "E2"],
  "D2": ["C2", "D1", "D3", "E2"],
  "D3": ["C3", "D2", "E2"],

  // Row 5 (center)
  "E1": ["A1", "E2", "I1"],
  "E2": ["D1", "D2", "D3", "E1", "E3", "F1", "F2", "F3"],
  "E3": ["A3", "E2", "I3"],

  // Row 6
  "F1": ["E2", "F2", "G1"],
  "F2": ["E2", "F1", "F3", "G2"],
  "F3": ["E2", "F2", "G3"],

  // Row 7
  "G1": ["F1", "G2", "H1"],
  "G2": ["F2", "G1", "G3", "H2"],
  "G3": ["F3", "G2", "H3"],

  // Row 8
  "H1": ["G1", "H2", "I1"],
  "H2": ["G2", "H1", "H3", "I2"],
  "H3": ["G3", "H2", "I3"],

  // Row 9
  "I1": ["E1", "H1", "I2"],
  "I2": ["H2", "I1", "I3"],
  "I3": ["E3", "H3", "I2"]
};
```

---

## Appendix B: Node Coordinate Mapping

Visual coordinates for rendering (adjust based on canvas size):

```javascript
const nodePositions = {
  "A1": {x: 150, y: 50},   "A2": {x: 400, y: 50},   "A3": {x: 650, y: 50},
  "B1": {x: 170, y: 120},  "B2": {x: 400, y: 120},  "B3": {x: 630, y: 120},
  "C1": {x: 190, y: 190},  "C2": {x: 400, y: 190},  "C3": {x: 610, y: 190},
  "D1": {x: 210, y: 260},  "D2": {x: 400, y: 260},  "D3": {x: 590, y: 260},
  "E1": {x: 230, y: 330},  "E2": {x: 400, y: 330},  "E3": {x: 570, y: 330},
  "F1": {x: 210, y: 400},  "F2": {x: 400, y: 400},  "F3": {x: 590, y: 400},
  "G1": {x: 190, y: 470},  "G2": {x: 400, y: 470},  "G3": {x: 610, y: 470},
  "H1": {x: 170, y: 540},  "H2": {x: 400, y: 540},  "H3": {x: 630, y: 540},
  "I1": {x: 150, y: 610},  "I2": {x: 400, y: 610},  "I3": {x: 650, y: 610}
};
```

---

## End of Planning Document

This plan serves as the complete blueprint for implementing Warp Lines. Follow the phases sequentially, test thoroughly at each stage, and refer back to this document for clarification on rules and requirements.
