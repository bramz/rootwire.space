---
author: "Brock Ramsey"
date: 2018-03-12
linktitle: Tic Tac Toe in Javscript
menu:
  main:
    parent: tutorials
title: Tic Tac Toe in Javascript
weight: 10
---
This walkthrough/tutorial will show you how to create a simple stand-alone tic tac toe application in javascript. We will assume that you are familiar with javascript and have used node.
I will explain how to create a simple stand-alone tic tac toe application with no user input. One will need to implement user input support to suit own needs.

## Getting Started
We have the game of tic tac toe, which consists of two players. Each player uses X or O to make a move on the board. We will assume the first move starts with 'X' and ends with 'X'. We have a 3x3 grid for the board. Leaving us with 9 total spaces on the board, consider the following.

```
[ ' ', ' ', ' ']
[ ' ', ' ', ' ']
[ ' ', ' ', ' ']
```
First, we will want to declare constants for handling turns and game state. This will help us handle who's turn it is and the current state of the game.  Proceeded by a declaration of the board using a multidimensional array.
```
const X_TURN = 0
const O_TURN = 1
const GAME_END = 2

const gameStatus = X_TURN
const board = [
    [' ', ' ', ' '],
    [' ', ' ', ' '],
    [' ', ' ', ' ']
]
```
## Handling Win Scenarios
If you count each win scenario on a 3x3 board, we are left with 8 possible outcomes. We must create a function to handle each of these scenarios and return boolean according to each won state. Please note, this can most likely be done in a more efficient manner. (Maybe try an alternative method for checking wins?)

```
function hasPlayerWon(player) {
  if (board[0][0] === player && board[1][0] === player && board[2][0] === player) { // row left up/down
    return true;
  } else if (board[0][0] === player && board[0][1] === player && board[0][2] === player) { // row top left/right
    return true;
  } else if (board[0][1] === player && board[1][1] === player && board[2][1] === player) { // row middle up/down
    return true;
  } else if (board[0][2] === player && board[1][2] === player && board[2][2] === player) { // row right up/down
    return true;
  } else if (board[1][0] === player && board[1][1] === player && board[1][2] === player) { // row middle left/right
    return true;
  } else if (board[2][0] === player && board[2][1] === player && board[2][2] === player) { // row bottom left/right
    return true;
  } else if (board[0][0] === player && board[1][1] === player && board[2][2] === player) { // row diagnonal top left/bottom right
    return true;
  } else if (board[0][2] === player && board[1][1] === player && board[2][0] === player) { // diagonal bottome left/top right
    return true;
  } else {
    return false;
  }
}
```
## Checking the Board
What if the board is full or not? We will want to create a function to check if the board is full or not. This will also help us determine if there is a draw or not. Check for a blank space on the board, if space exists return false boolean, the board is not full. Otherwise, we return true as the board is full.
We use a nested for loop to iterate the board spaces and perform a check for a blank string.
```
function isBoardFull() {
  let string = ' ';
  let index = [];
  for (let i=0; i < board.length; i++) {
    for (let k=0; k < board[i].length; k++) {
      if (board[i][k] === string) {
        return false;
      }
    }
  }
  return true;
}
```
## Performing Game Actions
Using a `while()` loop, we will finalize and handle the moves and responses to the moves. First we check `gameStatus`, if it is not equivalent to the `GAME_END` constant, we start the game. We check if the current status is set to `X_TURN`. After determining which state the game is in and who's turn it is, we check if the player has won using our function. If the player has won, we output that they have won and ``break`` the loop. If the player has not won, we check if the board is full or not. If the board is not full in `X`'s case we push the turn to `O`, if it is full, however, it is a draw. We perform the same actions for player `O`, however, we always start with player `X`. This means that the end move will never land on player `O`, so we do not need to handle a draw scenario during their turn. 
```
while (gameStatus != GAME_END) {
  if (gameStatus === X_TURN) {
    if (hasPlayerWon('X')) {
      console.log("player X has won!")
      break;
    } else {
      if (isBoardFull()) {
        gameStatus = O_TURN
      } else {
        console.log('Draw')
        break;
      }
    }
  } else if (gameStatus === O_TURN) {
    if (hasPlayerWon('O')) {
      console.log("player O has won!")
      break;
    } else {
      if (isBoardFull()) {
        gameStatus = X_TURN;
      }
    }
  }
}
```
Remember there is no user input here, thus meaning this is simply a skeleton for the game, you will need to handle user data accordingly. That sums it up, for a very basic model to assist with anyone trying to build a tic tac toe application.
