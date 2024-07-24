# codsoft_taskno.2
# Tic-Tac-Toe Game

## Overview
This project is a web-based Tic-Tac-Toe game that allows a user to play against the CPU. The game uses HTML, CSS, and JavaScript. The CPU utilizes the minimax algorithm to determine the optimal move.

## Features
- **Interactive UI**: Click on cells to make a move.
- **CPU Moves**: Let the CPU make a move by clicking a button or wait for the CPU to play its turn after the user.
- **New Game**: Start a new game at any time.

## Installation
1. Clone the repository:
    ```bash
    git clone <repository-url>
    cd <repository-directory>
    ```

2. Open `index.html` in a web browser to start playing the game.

## Usage
1. **Start a New Game**: Click the "New Game" button to reset the board and start a new game.
2. **Make a Move**: Click on an empty cell to make your move.
3. **CPU Move**: Click the "Let CPU play a move" button to let the CPU make a move if it's the CPU's turn.

## Code Explanation

### HTML
The HTML file sets up the basic structure of the game, including a table for the game board, message display, and buttons for starting a new game and letting the CPU make a move.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tic-Tac-Toe</title>
    <link rel="stylesheet" href="index.css">
</head>
<body>
    <table id="game">
        <tr><td></td><td></td><td></td></tr>
        <tr><td></td><td></td><td></td></tr>
        <tr><td></td><td></td><td></td></tr>
    </table>
    <h4 id="message"></h4>
    <button id="newgame">New Game</button>
    <button id="cpumove">Let CPU play a move</button>
    <script src="script.js"></script>
</body>
</html>
```

### CSS
The CSS file styles the game board and elements to provide a user-friendly interface.

```css
body{
    display: flex;
    flex-direction: column;
    justify-items: center;
    align-items: center;
}
#game { 
    border-collapse: collapse;
    display:inline-block;
    font-family: Arial, Helvetica, sans-serif;
}
#game td { 
    border: 1px solid black; 
    width: 100px; height: 100px; 
    text-align: center; 
    cursor: pointer; 
    font-size: 50px; 
} 
#game td.X, #game td.O { 
    cursor: default; 
}
#game td.X { 
    color: green; 
}
#game td.O { 
    color: red; 
}
#game td.X:after { 
    content: "X"; 
}
#game td.O:after { 
    content: "O"; 
}
#game.inactive { 
    background: silver; 
}
```

### JavaScript
The JavaScript file contains the game logic, including the Tic-Tac-Toe class, which handles the game state, and functions for user and CPU moves.

```javascript
class TicTacToe {
    constructor() {
        this.board = Array(9).fill(0); // 0 means "empty"
        this.moves = [];
        this.isWin = this.isDraw = false;
    }
    get turn() { // returns 1 or 2
        return 1 + this.moves.length % 2;
    }
    get validMoves() {
        return [...this.board.keys()].filter(i => !this.board[i])
    }
    play(move) { // move is an index in this.board
        if (this.board[move] !== 0 || this.isWin) return false; // invalid move
        this.board[move] = this.turn; // 1 or 2
        this.moves.push(move);
        // Use regular expression to detect any 3-in-a-row
        this.isWin = /^(?:...)*([12])\1\1|^.?.?([12])..\2..\2|^([12])...\3...\3|^..([12]).\4.\4/.test(this.board.join(""));
        this.isDraw = !this.isWin && this.moves.length === this.board.length;
        return true;
    }
    takeBack() {
        if (this.moves.length === 0) return false; // cannot undo
        this.board[this.moves.pop()] = 0;
        this.isWin = this.isDraw = false;
        return true;
    }
    minimax() {
        if (this.isWin) return { value: -10 };
        if (this.isDraw) return { value: 0 };
        let best = { value: -Infinity };
        for (let move of this.validMoves) {
            this.play(move);
            let {value} = this.minimax();
            this.takeBack();
            // Reduce magnitude of value (so shorter paths to wins are prioritised) and negate it
            value = value ? (Math.abs(value) - 1) * Math.sign(-value) : 0;
            if (value >= best.value) {
                if (value > best.value) best = { value, moves: [] };
                best.moves.push(move); // keep track of equally valued moves
            }
        }
        return best;
    }
    goodMove() {
        let {moves} = this.minimax();
        // Pick a random move when there are choices:
        return moves[Math.floor(Math.random() * moves.length)];
    }
}

(function main() {
    const table = document.querySelector("#game");
    const btnNewGame = document.querySelector("#newgame");
    const btnCpuMove = document.querySelector("#cpumove");
    const messageArea = document.querySelector("#message");
    let game, human;

    function display() {
        game.board.forEach((cell, i) => table.rows[Math.floor(i / 3)].cells[i % 3].className = " XO"[cell]);
        messageArea.textContent = game.isWin ? (game.turn == human ? "CPU won" : "You won")
                                : game.isDraw ? "It's a draw"
                                : game.turn == human ? "Your turn" 
                                : "CPU is preparing move...";
        table.className = game.isWin || game.isDraw || game.turn !== human ? "inactive" : "";
    }

    function computerMove() {
        if (game.isWin || game.isDraw) return; 
        human = 3 - game.turn;
        display();
        setTimeout(() => {
            game.play(game.goodMove());
            display();
        }, 500); // Artificial delay before computer move is calculated and played
    }

    function humanMove(i) {
        if (game.turn !== human || !game.play(i)) return; // ignore click when not human turn, or when invalid move
        display();
        computerMove();
    }

    function newGame() {
        game = new TicTacToe();
        human = 1;
        display();
    }

    table.addEventListener("click", e => humanMove(e.target.cellIndex + 3 * e.target.parentNode.rowIndex));
    btnNewGame.addEventListener("click", newGame);
    btnCpuMove.addEventListener("click", computerMove);
    newGame();
})();
```

## Contributing
Feel free to fork this project and submit pull requests. For major changes, please open an issue first to discuss what you would like to change.

## Acknowledgements
- [HTML](https://developer.mozilla.org/en-US/docs/Web/HTML)
- [CSS](https://developer.mozilla.org/en-US/docs/Web/CSS)
- [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
