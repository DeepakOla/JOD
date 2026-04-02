# Warp Lines - Quick Start Guide

## What is Warp Lines?

Warp Lines is a kid-friendly abstract strategy board game inspired by checkers, but played on a unique graph-based board. It's designed for children ages 7-12 to develop strategic thinking, spatial reasoning, and turn-taking skills.

## Quick Start

### Play Immediately
1. Open `warp-lines.html` in any modern web browser
2. No installation required - it's a single self-contained file!
3. Works on desktop and mobile devices

### Game Modes
- **Player vs Player (PvP)**: Two people play on the same device
- **Player vs Computer (PvC)**: Play against a simple AI opponent

## Basic Rules

### Setup
- **Red** starts at the top with 13 pieces
- **Green** starts at the bottom with 13 pieces
- **Center position** (E2) starts empty

### How to Play
1. **Click your piece** to select it
2. **Click where to move** (highlighted nodes)
3. **Take turns** with your opponent

### Movement Types

#### Normal Step
- Move to an adjacent empty node
- Follow the lines on the board

#### Capture (Jump)
- Jump over an enemy piece to capture it
- The captured piece is removed
- If you can capture again, you MUST continue jumping!

### The Golden Rule: MANDATORY CAPTURE
⚠️ **If ANY of your pieces can capture, you MUST make a capture move!**
- You cannot make a normal step when a capture is available
- This is the most important rule to remember

### Winning
- Eliminate all opponent pieces, OR
- Block opponent so they have no legal moves

## File Structure

```
JOD/
├── README.md              # Main project readme
├── warp-lines.html        # The complete game (open this to play!)
├── WARP_LINES_PLAN.md     # Complete implementation documentation
├── TESTING.md             # Test results and validation
└── QUICKSTART.md          # This file
```

## Technical Details

### Board Structure
- **27 nodes** arranged in 9 rows × 3 columns
- **Graph-based** - not a square grid
- Nodes are labeled A1-A3, B1-B3, ..., I1-I3
- **E2** is the central hub connecting to 8 nodes

### Implemented Features
✅ Complete game rules with mandatory capture
✅ Multi-jump capture sequences
✅ Simple AI opponent with strategy
✅ Smooth animations
✅ Win detection (elimination & blockade)
✅ Responsive design (desktop & mobile)
✅ Touch and mouse input
✅ In-game rules reference

### Code Quality
- **Single file**: Everything in warp-lines.html
- **No dependencies**: Pure HTML/CSS/JavaScript
- **File size**: 32KB
- **Well-commented**: Easy to understand and modify
- **No errors**: Clean console, no warnings

## For Developers

### Understanding the Code

The game is organized into clear sections:

1. **Configuration** - Colors, sizes, constants
2. **Board Topology** - Node adjacency list (the graph)
3. **Game State** - Current board, player, selections
4. **Move Generation** - Finding legal moves
5. **Move Execution** - Applying moves to board
6. **AI Logic** - Computer opponent strategy
7. **Rendering** - Canvas drawing functions
8. **Input Handling** - Click/touch detection
9. **UI Management** - Status, buttons, modals

### Key Functions

```javascript
// Game initialization
initGame()                    // Set up new game
resetGame()                   // Alias for initGame()

// Move generation
getLegalMovesForPiece(nodeId) // Get moves for one piece
getAllLegalMoves(player)      // Get all moves for player
getCaptureMovesForPiece(...)  // Find capture sequences
updateMustCaptureFlag()       // Check mandatory capture

// Move execution
executeMove(move)             // Perform move with animation
animateSimpleMove(...)        // Animate normal move
animateCaptureSequence(...)   // Animate multi-jump

// AI
aiMove()                      // Computer makes move
evaluateMove(move, player)    // Score move quality

// Rendering
drawBoard()                   // Draw entire game state
drawPieceAt(x, y, color)      // Draw single piece

// Input
handleCanvasClick(event)      // Process player clicks
getNodeAtPosition(x, y)       // Hit detection
```

### Modifying the Game

#### Change Colors
Edit the `COLORS` constant at the top:
```javascript
const COLORS = {
    red: '#e74c3c',    // Change red piece color
    green: '#2ecc71',  // Change green piece color
    // ... etc
};
```

#### Adjust AI Difficulty
Modify the `evaluateMove()` function weights:
```javascript
// Higher values = more important
score += move.captured.length * 1000;  // Capture value
score += (myMobility - oppMobility) * 10;  // Mobility value
```

#### Change Animation Speed
Edit animation duration:
```javascript
const duration = 300;  // milliseconds per move segment
```

### DO NOT Modify
⚠️ **Never change these without careful review:**
- The `neighbors` adjacency list (defines board topology)
- Starting positions (13 pieces each, specific nodes)
- Capture logic (A-B-C pattern validation)
- Mandatory capture check

## Educational Value

### Cognitive Skills
- **Strategic Thinking**: Plan ahead, anticipate opponent
- **Spatial Reasoning**: Understanding graph connectivity
- **Pattern Recognition**: Spotting capture opportunities
- **Problem Solving**: Finding optimal move sequences

### Social Skills
- **Turn-Taking**: Patience and waiting
- **Fair Play**: Following rules consistently
- **Handling Outcomes**: Winning and losing gracefully
- **Communication**: Discussing moves and strategy

### Mathematical Concepts
- **Graph Theory**: Nodes, edges, connectivity
- **Combinatorics**: Counting possible moves
- **Logic**: If-then reasoning
- **Geometry**: Spatial relationships

## Troubleshooting

### Game Won't Load
- Ensure you're using a modern browser (Chrome 90+, Firefox 88+, Safari 14+)
- Check JavaScript is enabled
- Try opening the file directly (file:// URL is fine)

### Pieces Won't Move
- Make sure it's your turn (check status bar)
- Verify you're clicking your own pieces
- If "MUST CAPTURE!" shows, you can only make capture moves

### Can't Capture When Should
- Verify the landing node (beyond enemy) is empty
- Check that there's a valid edge path: your piece → enemy → landing
- Remember: can only jump over ONE piece at a time

### AI Not Responding
- Wait a moment (AI thinks for ~500ms)
- Check browser console for errors (F12)
- Ensure you're in PvC mode (not PvP)

### Mobile Issues
- Make sure touch targets are being registered
- Try tapping piece center, not edges
- Disable browser zoom/scroll if interfering

## Future Enhancements

### Planned Features (Not Yet Implemented)
- [ ] Sound effects (move, capture, win)
- [ ] Multiple AI difficulty levels
- [ ] Undo move functionality
- [ ] Move history display
- [ ] Save/load game state
- [ ] Statistics tracking
- [ ] Hint system
- [ ] Tutorial mode
- [ ] Draw conditions (repetition, no capture)
- [ ] Online multiplayer (requires backend)

### How to Add Features

Want to contribute? Here's where to start:

1. **Sound Effects**: Add `<audio>` elements, play on move/capture
2. **Undo**: Store move history array, implement reverse operations
3. **Hints**: Use AI `evaluateMove()` to suggest best move
4. **Stats**: Add localStorage to save wins/losses/games played
5. **Themes**: Create alternate color schemes, add switcher

See `WARP_LINES_PLAN.md` for detailed feature roadmap.

## Support & Feedback

### Getting Help
1. Read this guide thoroughly
2. Check `WARP_LINES_PLAN.md` for technical details
3. Review `TESTING.md` for known behaviors
4. Examine code comments in warp-lines.html

### Reporting Issues
If you find a bug:
1. Describe what happened vs what you expected
2. Include steps to reproduce
3. Note your browser/device
4. Check console for errors (F12 → Console tab)

### Suggesting Improvements
Ideas welcome! Consider:
- Does it fit the "kid-friendly" theme?
- Can it be done in a single HTML file?
- Does it make the game more fun or educational?

## Credits

**Game Design**: Based on abstract strategy games like Checkers/Draughts
**Implementation**: Claude Code (Anthropic)
**Project**: JOD (Journey of Delight) - Nostalgia Games Collection

## License

This is a hobby/educational project. Feel free to:
- Play and share the game
- Learn from the code
- Modify for personal use
- Use as a teaching example

---

## Quick Reference Card

### Controls
- **Click/Tap**: Select piece or move destination
- **New Game**: Reset game to start
- **Mode Toggle**: Switch PvP ↔ PvC
- **Rules**: Open rules reference

### Win Conditions
- Eliminate all opponent pieces
- Block opponent (no legal moves)

### Key Rule
⚠️ **MUST CAPTURE** if able!

### File to Play
`warp-lines.html` ← Open this in your browser!

---

**Enjoy the game! May the best strategist win! 🎮**
