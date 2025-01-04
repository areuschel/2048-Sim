# 2048-Sim
Simulation study testing 2048 game outcomes with varying strategies and new tile generation probabilities.

## Project Description

The game 2048 was a staple of my childhood, one of the few non-blocked games I could play on a middle school Chromebook. I still play the game today, using it often to pass time during long commutes in the city. Although the game's primary goal is the 2048 tile, I've been able to reach the 4096 tile, and further iterations are possible.

My proof...

![break](/Photos/highscore.png?raw=true "Break")

#### 📍 Study Goal
My simulation study aims to learn how different strategies perform as the probability of generating a new tile with the value ‘2’ changes. I explore 4 strategies and compare their performance at each tile generation setting by investigating the game score, final board, and total number of moves made.
- See 'About 2048' section below to learn the rules if you are unfamiliar with the game.

## Language & Utilities
- RStudio
- tidyverse
- survival

## Project Walk-Through

### Table of Contents

![break](/Photos/sp1.jpg?raw=true "Break")

- About 2048
  - Game Origin
  - Rules and Playing the Game
- Coding the Game Functionalities in R
  - Generate Starting Board
  - Horizontal and Lateral Movements with Merges
  - New Tile Random Generation
- Implemented Strategies for Simulation Study
- Simulating Games of 2048
- Results
  - Estimated Densities
    - Game Score
    - Total Moves
  - Relationship between Moves and Score
  - Kaplan-Meyer Survival Curve
- Conclusion and Future Considerations

## About 2048

### Game Origin

### Rules and Playing the Game

#### Basic Understanding
The game is played on a 4x4 tile board, where tile numbers increase as identical numbers are combined. The goal is to combine tiles to create the number '2048'. Each move adds another tile to the board, so you need to be combining tiles frequently enough the board doesn't fill with tiles that cannot be combined.

#### Playing the Game
1. START
    - Two tiles with the value ‘2’ spawn in random places to start the game
2. GAMEPLAY
    - Move ALL tiles in any cardinal direction by swiping (mobile version) or with ⬆️ ⬇️ ➡️ ⬅️ arrows (PC)
    - A new tile is randomly generated each time you move in any of the available spaces on the board
      - This new tile will either be a '2' or a '4'
        - The default settings of the game have the probability of generating a ‘2’ at 90% and generating a ‘4’ at 10% for new tiles
    - Tiles that are of the same number will be combined (if any!) in the direction chosen
      - Tiles will be combined furthest to the direction chosen
        - Example row) 2 2 2 4
          - chosen move = ➡️
          - outcome = NA 2 4 4
4. END
   - The game is over when the entire board is ...
     - (1) full of tiles AND
     - (2) none of the tiles can be combined by a move in any direction


## Coding the Game Functionalities in R

![spain](/Photos/sp2.JPG?raw=true "spain")

### Generate Starting Board

### Horizontal and Lateral Movements with Merges

### New Tile Random Generation

### Implemented Strategies for Simulation Study
First, a strategy where each move is chosen randomly. This was the simplest to begin with. I then test two repetitive strategies: one where the preferred move sequence is ‘down-left-down-left’ and another that switches between ‘left-right-left-right’ and ‘up-down-up-down’. For both of these strategies, if the next move in the sequence doesn’t change the board, a random move is selected to continue the game before returning to the original sequence. Lastly, I attempt to simplify my own strategy to the game.

### Simulating Games of 2048

![break](/Plots/sim2048.JPEG?raw=true "Break")

## Results

### Estimated Densities

![break](/Plots/comp1.JPEG?raw=true "Break")

![break](/Plots/comp2.JPEG?raw=true "Break")

### Relationship between Moves and Score

![break](/Plots/scatter.JPEG?raw=true "Break")

### Kaplan-Meyer Survival Curve

![break](/Plots/surv.JPEG?raw=true "Break")

## Conclusion and Future Considerations

![Adri](/Photos/zumaya.jpeg?raw=true "Zumaya")

