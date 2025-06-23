2048 is a classic puzzle game where players slide the screen to merge blocks with the same numbers, with the ultimate goal of synthesizing the number 2048. This article is based on the HarmonyOS ArkUI framework and provides a detailed analysis of its implementation process, explaining how to use declarative UI and state management to build such games.

![](https://cdn.nlark.com/yuque/0/2025/jpeg/12749040/1741336965425-3bbbfd3e-c9d6-45f5-86d9-7fb72e994dd8.jpeg?x-oss-process=image%2Fformat%2Cwebp)

### I. Core Data Structures and State Management
#### 1. Game Grid and Scores
The core of the game is a 4x4 two-dimensional array used to store the numbers in each grid. The grid state is managed through the @State decorator to ensure that the UI is automatically refreshed when the data changes:

```typescript
@State grid: number[][] = Array(4).fill(0).map(() => Array(4).fill(0));
@State score: number = 0; // Current score
@State bestScore: number = 0; // Historical highest score
```

#### 2. Game Initialization
The initGame method is responsible for resetting the grid, adding initial blocks, and resetting the score. addNewTile is used to generate new blocks in random empty positions (90% probability of generating 2, 10% probability of generating 4):

```typescript
initGame() {
  this.grid = this.grid.map(() => Array(4).fill(0));
  this.addNewTile();
  this.addNewTile();
  this.score = 0;
}
```

### II. Sliding Logic and Merging Algorithm
#### 1. Direction Handling and Matrix Rotation
The game supports sliding in four directions: up, down, left, and right. To simplify the code logic, matrix rotation is used to uniformly convert movements in different directions into leftward movements:

+ **Leftward**: Directly process each row.
+ **Rightward**: Reverse the row, move left, and then reverse it back.
+ **Upward/Downward**: Rotate the matrix into rows, process it, and then restore it to columns.

```typescript
// Matrix rotation helper method
const rotate = (matrix: number[][]): number[][] => {
  return matrix[0].map((_, i) => matrix.map(row => row[i]).reverse());
};
```

#### 2. Single-Row Merging Logic
The processing of each row is divided into three steps:

1. **Remove Spaces**: Filter out non-zero numbers.
2. **Merge Identical Numbers**: Merge adjacent identical numbers and accumulate the score.
3. **Complete Length**: Fill with zeros to a length of 4.

```typescript
const moveRow = (row: number[]): number[] => {
  let newRow = row.filter(cell => cell !== 0);
  for (let i = 0; i < newRow.length - 1; i++) {
    if (newRow[i] === newRow[i + 1]) {
      newRow[i] *= 2;
      this.score += newRow[i]; // Score accumulation
      newRow.splice(i + 1, 1);
    }
  }
  return [...newRow, ...Array(4 - newRow.length).fill(0)];
};
```

### III. Game End Judgment
The game ends when the grid is full and there are no adjacent blocks that can be merged. The detection is carried out through the following steps:

1. **Check for Spaces**: If there are spaces, the game has not ended.
2. **Horizontal Detection**: Traverse each row to check for adjacent identical numbers.
3. **Vertical Detection**: Traverse each column to check for adjacent identical numbers.

```typescript
isGameOver(): boolean {
  if (this.grid.some(row => row.includes(0))) return false;
  // Horizontal and vertical detection logic
  // ...
  return true;
}
```

### IV. UI Implementation and Interaction Design
#### 1. Grid Rendering
The Grid component is used to dynamically generate a 4x4 grid, and each GridItem displays different background colors and text colors according to the numerical value:

```typescript
Grid() {
  ForEach(this.grid, (row: number[], i) => {
    ForEach(row, (value: number, j) => {
      GridItem() {
        Text(value ? `${value}` : '')
          .backgroundColor(this.getTileColor(value))
          .fontColor(this.getTextColor(value));
      }
    })
  })
}
```

#### 2. Touch Event Handling
The onTouch event listens for sliding events, calculates the difference between the start and end coordinates, and determines the sliding direction:

```typescript
onTouch((event) => {
  if (event.type === TouchType.Down) {
    this.startX = event.touches[0].x;
    this.startY = event.touches[0].y;
  } else if (event.type === TouchType.Up) {
    const deltaX = event.touches[0].x - this.startX;
    const deltaY = event.touches[0].y - this.startY;
    // Judge the direction and call the move method
  }
});
```

### V. Local Storage and Animation Effects
#### 1. High Score Persistence
PreferencesUtil is used to store and read the highest score to ensure that the data is retained after the application is restarted:

```typescript
aboutToAppear() {
  this.bestScore = PreferencesUtil.getNumberSync("bestScore");
}

// Update the highest score
if (this.score > this.bestScore) {
  PreferencesUtil.putSync('bestScore', this.score);
}
```

#### 2. Animation and Visual Effects
Each block's text change is added with a 150ms gradient animation to enhance the user experience:

```typescript
Text(value ? `${value}` : '')
  .animation({ duration: 150, curve: Curve.EaseOut });
```

### VI. Summary and Complete Code
Through ArkUI's declarative UI and state management, the core logic of 2048 can be efficiently implemented. The key points include:

+ **Matrix rotation** simplifies direction processing.
+ **State-driven** UI automatic update.
+ Smooth combination of touch events and animations.

```typescript
import { HashMap } from '@kit.ArkTS'
import { AppUtil, PreferencesUtil, ToastUtil } from '@pura/harmony-utils'

// index.ets
@Entry
@Component
struct Game2048 {
  @State grid: number[][] = Array(4).fill(0).map(() => Array(4).fill(0)) // 4x4 game grid
  @State score: number = 0 // Current score
  @State bestScore: number = 0 // Historical highest score
  private startX: number = 0 // Touch start X coordinate
  private startY: number = 0 // Touch start Y coordinate

  // Lifecycle method: triggered when the page is about to be displayed
  aboutToAppear() {
    this.initGame()
    this.bestScore = PreferencesUtil.getNumberSync("bestScore") // Read the highest score stored locally
  }

  // Initialize the game
  initGame() {
    this.grid = this.grid.map(() => Array(4).fill(0)) // Reset the grid
    this.addNewTile() // Add two new blocks
    this.addNewTile() // Reset the current score
    this.score = 0
  }

  addNewTile() {
    const emptyCells: [number, number][] = [] // Collect the coordinates of empty cells
    this.grid.forEach((row, i) => {
      row.forEach((cell, j) => {
        if (cell === 0) {
          emptyCells.push([i, j])
        }
      })
    })

    if (emptyCells.length > 0) {
      let n = Math.floor(Math.random() * emptyCells.length) // Randomly select an empty cell
      const i = emptyCells[n][0]
      const j = emptyCells[n][1]
      this.grid[i][j] = Math.random() < 0.9 ? 2 : 4 // 90% probability of generating 2, 10% probability of generating 4
    }
  }

  // Process movement logic
  move(direction: 'left' | 'right' | 'up' | 'down') {
    let newGrid = this.grid.map(row => [...row]) // Create a grid copy
    let moved = false // Movement flag

    // Matrix rotation helper methods
    const rotate = (matrix: number[][]): number[][] => {
      return matrix[0].map((_, i) => matrix.map(row => row[i]).reverse())
    }
    const rotateReverse = (matrix: number[][]): number[][] => {
      return matrix[0].map((_, i) => matrix.map(row => row[row.length - 1 - i]))
    }

    // Process single-row movement and merging
    const moveRow = (row: number[]): number[] => {
      let newRow = row.filter(cell => cell !== 0) // Remove spaces
      for (let i = 0; i < newRow.length - 1; i++) {
        if (newRow[i] === newRow[i + 1]) { // Merge identical numbers
          newRow[i] *= 2
          this.score += newRow[i] // Update the score
          newRow.splice(i + 1, 1) // Remove the merged element
        }
      }

      // Complete the length
      while (newRow.length < 4) {
        newRow.push(0)
      }
      return newRow
    }

    // Process movement according to direction
    switch (direction) {
      case 'left':
        newGrid.forEach((row, i) => newGrid[i] = moveRow(row))
        break
      case 'right':
        newGrid.forEach((row, i) => newGrid[i] = moveRow(row.reverse()).reverse())
        break
      case 'up':
        let rotatedDown = rotate(newGrid)
        rotatedDown.forEach((row, i) => rotatedDown[i] = moveRow(row.reverse()).reverse())
        newGrid = rotateReverse(rotatedDown)
        break
      case 'down':
        let rotatedUp = rotate(newGrid)
        rotatedUp.forEach((row, i) => rotatedUp[i] = moveRow(row))
        newGrid = rotateReverse(rotatedUp)
        break
    }

    moved = JSON.stringify(newGrid) !== JSON.stringify(this.grid) // Judge whether movement has occurred
    this.grid = newGrid

    if (moved) {
      this.addNewTile() // Add a new block after movement
      if (this.score > this.bestScore) { // Update the highest score
        this.bestScore = this.score
        PreferencesUtil.putSync('bestScore', this.bestScore) // Save the highest score
      }
    }

    if (this.isGameOver()) { // Game end detection
      ToastUtil.showToast('Game Over!')
    }
  }

  // Game end judgment
  isGameOver(): boolean {
    // Check for empty cells
    if (this.grid.some(row => row.includes(0))) {
      return false
    }

    // Check for horizontal merges
    for (let i = 0; i < 4; i++) {
      for (let j = 0; j < 3; j++) {
        if (this.grid[i][j] === this.grid[i][j + 1]) {
          return false
        }
      }
    }

    // Check for vertical merges
    for (let j = 0; j < 4; j++) {
      for (let i = 0; i < 3; i++) {
        if (this.grid[i][j] === this.grid[i + 1][j]) {
          return false
        }
      }
    }

    return true
  }

  build() {
    Column() {
      // Score display row
      Row() {
        Text(`Score: ${this.score}`)
          .fontSize(20)
          .margin(10)
        Text(`Highest Score: ${this.bestScore}`)
          .fontSize(20)
          .margin(10)
        Button('New Game')
          .onClick(() => this.initGame())
          .margin(10)
      }.margin({ top: px2vp(AppUtil.getStatusBarHeight()) })

      // Game grid
      Grid() {
        ForEach(this.grid, (row: number[], i) => {
          ForEach(row, (value: number, j) => {
            GridItem() {
              Text(value ? `${value}` : '')
                .textAlign(TextAlign.Center)
                .fontSize(24)
                .fontColor(this.getTextColor(value))
                .width('100%')
                .height('100%')
                .backgroundColor(this.getTileColor(value))
                .animation({
                  duration: 150,
                  curve: Curve.EaseOut
                })
            }.key(`${i}-${j}`)
          })
        })
      }
      .columnsTemplate('1fr 1fr 1fr 1fr')    // 4 equal columns
      .rowsTemplate('1fr 1fr 1fr 1fr')       // 4 equal rows
      .width('90%')
      .aspectRatio(1)                        // Maintain square shape
      .margin(10)
      .onTouch((event) => {                  // Touch event handling
        if (event.type === TouchType.Down) {
          this.startX = event.touches[0].x
          this.startY = event.touches[0].y
        } else if (event.type === TouchType.Up) {
          const deltaX = event.touches[0].x - this.startX
          const deltaY = event.touches[0].y - this.startY

          // Judge the movement according to the sliding direction
          if (Math.abs(deltaX) > Math.abs(deltaY)) {
            deltaX > 0 ? this.move('right') : this.move('left')
          } else {
            deltaY > 0 ? this.move('down') : this.move('up')
          }
        }
      })
    }
    .width('100%')
  }

  // Get the block background color
  getTileColor(value: number): string {
    const colors = new HashMap<number, string>()
    colors.set(0, '#CDC1B4')
    colors.set(2, '#EEE4DA')
    colors.set(4, '#EDE0C8')
    colors.set(8, '#F2B179')
    colors.set(16, '#F59563')
    colors.set(32, '#F67C5F')
    colors.set(64, '#F65E3B')
    colors.set(128, '#EDCF72')
    colors.set(256, '#EDCF72')
    colors.set(512, '#EDCC61')
    colors.set(1024, '#EDC850')
    colors.set(2048, '#EDC22E')
    return colors.get(value) || '#CDC1B4'
  }

  // Get the text color
  getTextColor(value: number): Color {
    return value > 4 ? Color.White : Color.Black
  }
}
```

