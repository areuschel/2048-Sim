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

### 🕹️ Game Origin

Italian web developer Gabriele Cirulli created the game in 2014. It quickly gained popularity, reaching 4 million visitors in less than a week of its release. 

More variations of the game have been developed and are free to play online, including a version that lets you undo moves. However, I much prefer the original gameplay.

### 🕹️ Rules and Playing the Game

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

### 👾 Generate Starting Board

The code block below shows how I generated random starting boards for my simulation.

```{r}
# setup the board
board <- matrix(data = NA, nrow = 4, ncol = 4)

# randomly generate 2's in any space in the matrix:
# generate tiles without replacement from 1-16
# 1-4 = col 1
# 5-8 = col 2
# 9-12 = col 3
# 13-16 = col 4

initial <- sample(1:16, 2, replace = FALSE)

# convert sampled num's to matrix positions
# arrayInd()

init_board <- arrayInd(initial, .dim = c(4,4))

# place in board
board[init_board] = 2

# check
print(board)
```

### 👾 Horizontal and Lateral Movements

### ➡️
Function to move the tiles to the right.
```{r}
right <- function(board){
  for(row in 1:4){
    # find the tiles that contain numbers
    filled_tiles <- board[row, !is.na(board[row,])]
    
    # clear the current row
    board[row, ] <- NA
    
    # fill the cells closest to the right with the values found above
    if (length(filled_tiles) > 0) {
      start <- 4 - length(filled_tiles) + 1
      if (start >= 1 && start <= 4) {
        board[row, start:4] <- filled_tiles
      }
    }

    # merge values if identical (right to left if more than 2 in a row)
    for(col in 4:2){
      # check if they match
      if(!is.na(board[row,col]) && !is.na(board[row, col-1]) && board[row,col] == board[row, col-1]){
        board[row, col] <- board[row, col] * 2
        board[row, col - 1] <- NA
      }
    }
    # do the shift again
    filled_tiles <- board[row, !is.na(board[row,])]
    board[row, ] <- NA
    
    # fill the cells closest to the right
    if (length(filled_tiles) > 0) {
      start <- 4 - length(filled_tiles) + 1
      if (start >= 1 && start <= 4) {
        board[row, start:4] <- filled_tiles
      }
    }
  }
  return(board)
}
```
### ⬅️
Function to move the tiles to the left.
(Same logic as above, comments ommitted)
```{r}
left <- function(board){
  for(row in 1:4){
    filled_tiles <- board[row, !is.na(board[row,])]
    
    board[row, ] <- NA
    
    if (length(filled_tiles) > 0) {
      endr <- length(filled_tiles)
      if (endr >= 1 && endr <= 4) {
        board[row, 1:endr] <- filled_tiles
      }
    }

    for(col in 1:3){
      if(!is.na(board[row,col]) && !is.na(board[row, col+1]) && board[row,col] == board[row, col+1]){
        board[row, col] <- board[row, col] * 2
        board[row, col + 1] <- NA
      }
    }
    filled_tiles <- board[row, !is.na(board[row,])]
    board[row, ] <- NA
    
    if (length(filled_tiles) > 0) {
      endr <- length(filled_tiles)
      if (endr >= 1 && endr <= 4) {
        board[row, 1:endr] <- filled_tiles
      }
    }
  }
  return(board)
}
```

### ⬆️
Function to move the tiles upward.
```{r}
up <- function(board) {
  for (col in 1:4) {
    # find tiles in each column
    filled_tiles <- board[!is.na(board[, col]), col]
    board[, col] <- NA
    
    # fill tiles closest to the top with values found above
    if (length(filled_tiles) > 0) {
      endr <- length(filled_tiles)
      if (endr >= 1 && endr <= 4) {
        board[1:endr, col] <- filled_tiles
      }
    }
    
    # merge
    for (row in 1:3) {
      if (!is.na(board[row, col]) && !is.na(board[row + 1, col]) && board[row, col] == board[row + 1, col]) {
        board[row, col] <- board[row, col] * 2
        board[row + 1, col] <- NA
      }
    }
    
    # shift again
    filled_tiles <- board[!is.na(board[, col]), col]
    board[, col] <- NA
    if (length(filled_tiles) > 0) {
      endr <- length(filled_tiles)
      if (endr >= 1 && endr <= 4) {
        board[1:endr, col] <- filled_tiles
      }
    }
  }
  return(board)
}
```

### ⬇️
Function to move the tiles downward.
(Same logic as above, comments ommitted)

```{r}
down <- function(board) {
  for (col in 1:4) {
    filled_tiles <- board[!is.na(board[, col]), col]
    board[, col] <- NA
    if (length(filled_tiles) > 0) {
      endr <- length(filled_tiles)
      board[(5 - endr):4, col] <- filled_tiles
    }
    
    for (row in 3:1) {  # start at third row
      if (!is.na(board[row, col]) && !is.na(board[row + 1, col]) && board[row, col] == board[row + 1, col]) {
        board[row + 1, col] <- board[row, col] * 2  
        board[row, col] <- NA  # original tile to NA
      }
    }
    
    filled_tiles <- board[!is.na(board[, col]), col]
    board[, col] <- NA
    if (length(filled_tiles) > 0) {
      endr <- length(filled_tiles)
      board[(5 - endr):4, col] <- filled_tiles
    }
  }
  return(board)
}
```

### 👾 New Tile Random Generation

The function below takes a probability vector so that simulated games can test the outcomes using different probabilities. The game's default new tile generation is 90% '2' and 10% '4'.
```{r}
# game default ... p2=0.9
new_tile <- function(board, p_gen = p){
  # find open spaces
  open <- which(is.na(board), arr.ind = TRUE)
  # randomly select a space to fill
  if(nrow(open) > 0){
    selected <- open[sample(1:nrow(open), 1), ]
    # fill the space with 2 or 4
    fill_value <- ifelse(runif(1)<=p_gen, 2, 4)
    board[selected[1], selected[2]] <- fill_value
  }
  return(board)
}
```


### 🧩 Implemented Strategies for Simulation Study

I programmed 4 strategies to compare performance in simulated games against different new tile generation probabilities. The strategies are described below:

1. Random
    - Each move is randomly selected from the four movement functions.
3. Basic corner (AKA basic in data viz)
    - Preferred move sequence: down-left-down-left
    - If any move in the sequence doesn't change the board, a random move is selected before returning to the preferred sequence
5. Swap
    - Preferred move sequence: left-right, up-down 🔁
      - "swaps" from lateral and horizontal sequences
    - If any move in the sequence doesn't change the board, a random move is selected before returning to the preferred sequence
7. Adri (My simplified strategy)
    - Preferred move sequence: right-right, down-down
    - If any move in the sequence doesn't change the board, the next preferred move is up
      - If that doesn't change it either, moving left is the last resort



### 🧑🏼‍💻 Simulating Games of 2048

#### 1. Create DF to store outcomes
```{r}
# number of simulations (for each strategy @ each probability)
n_sim <- 2100

results_df <- data.frame(
  strat = character(),
  p_gen = numeric(),
  moves = integer(),
  score = integer(),
  final_board = I(list())
)
```

#### 2. Set a seed

```{r}
set.seed(2048)
```

#### 3. Set up probability vector and strategies
```{r}
# generation values
p_gen_values <- c(0, 0.1, 0.2, 0.3,0.4, 0.5, 0.6, 0.7, 0.8, 0.9, 1.0)

# strategy functions
strategies <- list(
  adri = adri_sim,  
  basic_corner = DL_sim,
  rando = random_sim,
  swap = swap_sim
)
```

#### 4. Simulate games
```{r}
for (p in p_gen_values) {
  for (strategy_name in names(strategies)) {
    strategy_function <- strategies[[strategy_name]]
    
    for (sim in 1:n_sim) {
      simulation_results <- strategy_function(p_gen = p)
      
      results_df <- rbind(
        results_df,
        data.frame(
          strat = strategy_name,
          p_gen = p,
          moves = simulation_results$`Total moves`,
          score = simulation_results$`Game score`,
          final_board = I(list(simulation_results$`Final board`))
        )
      )
    }
  }
}
```

#### 2048!

One of the simulated games reached the 2048 tile! The final board is shown below.

![break](/Plots/sim2048.JPEG?raw=true "Break")

## Results

### Estimated Densities

#### Game performance with new tile generation probabilities
![break](/Plots/comp1.JPEG?raw=true "Break")

#### Importance of random value generation
- The plots above reveal that the highest values for both performance metrics occur when randomness in value generation is eliminated (p=0 or p=1)
  - (* Note: not <b>all</b> randomness is eliminated—the empty tile which the selected value will fill is still selected at random)

#### Maximizing time
- The plot for expected number of moves has a defined U-shape
  - This minimum value falls somewhere around 0.2 or 0.3, and the expected values increase intuitively after this point since smaller values are more likely to generate--requiring more moves to combine tiles to higher values
- An argument for the default settings (p=0.9)
  - Maximizes the number of moves that can be made <b>without eliminating</b> one of the game’s two random variables (generation location & tile value)
    - This provides more opportunities to combine tiles, achieve higher scores, and prolong the game, enhancing its overall appeal and addictive nature


#### Game performance estimated kernal desnities
![break](/Plots/comp2.JPEG?raw=true "Break")

#### ❌ Worst strategies, based on the density plots above
- The ‘Swap’ strategy has a narrow distribution for both score and number of moves, peaking at low values for both
  - This means the strategy is consistent, but poor performing
- The ‘Random’ strategy exhibits a little more spread in both features, but still performs poorly overall

#### ✅ Best strategies, based on the density plots above
- The final two methods performed the best: the ‘Basic’—which prefers a sequence to force the tiles in a corner, and ‘Adri’— my  simplified strategy
  - My strategy has more spread and higher density bleeding into the high score ranges
    - This is a desirable outcome for generalizing a strategy since it more often allows the player to continue the game further in both moves and increasing score
    - <b>Additionally</b>, my ‘Adri’ strategy performs best for nearly all generation probabilities


### Relationship between Moves and Score

![break](/Plots/scatter.JPEG?raw=true "Break")

I like this plot because it shows that a game with 500 moves can result in a score as low as 1000 or, double that, 2000, putting more weight on the importance of strategy as you play the game rather than simply trying to swipe as long as possible.

### Kaplan-Meyer Survival Curve

![break](/Plots/surv.JPEG?raw=true "Break")

The insights from this plot are consistent with the other findings
- The ‘swap’ and ‘random’ strategies are much less likely to survive at each move than the basic corner strategy and my own. 
- For example, at 100 moves into a game, the probability of surviving for each strategy is estimated as follows:
  - Sswap(100) ≅ 0.2
  - Srandom(100) ≅ 0.37
  - Sbasic(100) ≅ 0.77
  - Sadri(100) ≅ 0.82
- This is plot shows more evidence that my simplified game strategy performs better than the basic corner strategy and that the random and swap methods do not lead to high game scores


## Conclusion and Future Considerations

![Adri](/Photos/zumaya.jpeg?raw=true "Zumaya")

