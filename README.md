<!DOCTYPE html>
<html>
<head>
    <title>Go Game</title>
</head>
<body>
<center><canvas id="board" width="500" height="500"></canvas>
    <br>
    <button id="confirm-button" onclick="confirmMove()">Confirm Move</button>
    <br>

</center>

<script>
		const canvas = document.getElementById("board");
		const ctx = canvas.getContext("2d");

        //Variables for button interaction things
        let lastX = 1;
        let lastY = 1;
        let playerTurn = 1;
        let board = [];
        const boardSize = 19;
        function initializeBoard(){
        for (let i = 0; i < boardSize; i++) {
            board.push([]);
            for (let j = 0; j < boardSize; j++) {
            board[i].push(0);
            }
        }
        }
        initializeBoard();
		// Draw the board
		ctx.fillStyle = "#f5d993";
		ctx.fillRect(0, 0, canvas.width, canvas.height);
		ctx.strokeStyle = "#000";
		ctx.lineWidth = 2;

		// Draw the intersections
		function drawIntersections(){
		const gridSize = 25; // Change this value to adjust the size of the board
		const numLines = canvas.width / gridSize;
		for (let i = 1; i < numLines; i++) {
			const pos = i * gridSize;
			ctx.beginPath();
			ctx.moveTo(pos, 25);
			ctx.lineTo(pos, canvas.height-25);
			ctx.stroke();

			ctx.beginPath();
			ctx.moveTo(25, pos);
			ctx.lineTo(canvas.width-25, pos);
			ctx.stroke();
		}
        }
        drawIntersections();
		// Draw the star points
        function drawStarPoints(){
		    const starPoints = [4, 10, 16]; // Change this array to adjust the position of the star points
		    const radius = 3; // Change this value to adjust the size of the star points
		    ctx.fillStyle = "#000";
		    const gridSize = 25;
		    for (let i = 0; i < starPoints.length; i++) {
		    	for (let j = 0; j < starPoints.length; j++) {
			    	const x = starPoints[i] * gridSize;
			    	const y = starPoints[j] * gridSize;
			    	ctx.beginPath();
			    	ctx.arc(x, y, radius, 0, 2 * Math.PI);
			    	ctx.fill();
			    }
		    }
		}
		drawStarPoints();
	    // Draw a piece at the specified x, y coordinates
        function drawPiece(x, y, color) {
            const gridSize = 25; // Change this value to match the board grid size
            const radius = 10; // Change this value to adjust the size of the pieces
            const centerX = (x * gridSize) + 25;
            const centerY = (y * gridSize) + 25;
            ctx.beginPath();
            ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI);
            ctx.fillStyle = color;
            ctx.fill();

            }
        // Remove the piece at the specified x, y coordinates

        function removePiece(x, y) {
            const gridSize = 25; // Change this value to match the board grid size
            ctx.fillStyle = "#f5d993"; // Fill the space with the board background color
            ctx.fillRect((x * gridSize) + 15, (y * gridSize) + 15, 20, 20);
            // Redraw the intersecting lines to cover up the removed piece
            ctx.strokeStyle = "#000";
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo((x * gridSize) + 25, (y * gridSize) + 15);
            ctx.lineTo((x * gridSize) + 25, (y * gridSize) + 35);
            ctx.stroke();
            ctx.beginPath();
            ctx.moveTo((x * gridSize) + 15, (y * gridSize) + 25);
            ctx.lineTo((x * gridSize) + 35, (y * gridSize) + 25);
            ctx.stroke();
            }
        // Add a button to each intersection on the board
        const buttons = new Map();
        function addButtons() {
            for (let i = 1; i < 20; i++) {
                for (let j = 1; j < 20; j++) {
                    addButton(i, j);
                }
            }
        }

        addButtons();
        function addButton(x, y){
            const gridSize = 25; // Change this value to match the board grid size
            const buttonSize = 20; // Change this value to adjust the size of the buttons
            const offset = (gridSize - buttonSize) / 2;
            const button = document.createElement("button");
            button.style.position = "absolute";
            button.style.left = (canvas.offsetLeft + (x + 4) * gridSize + offset - 113) + "px";
            button.style.top = (canvas.offsetTop + (y) * gridSize + offset - 13) + "px";
            button.style.width = buttonSize + "px";
            button.style.height = buttonSize + "px";
            button.style.borderRadius = "50%";
            button.style.border = "none";
            button.style.backgroundColor = "transparent";
            const id = `button-${x-1}-${y-1}`; // create a unique ID for each button
            button.id = id;
            button.addEventListener("click", function() {
                console.log("Button at position (" + x + ", " + y + ") was clicked!");

                lastX = x-1;
                lastY = y-1;
            });
            button.addEventListener("focus", function() {
                button.style.backgroundColor = "green";
            });
            button.addEventListener("blur", function() {
                button.style.backgroundColor = "transparent";
            });
            document.body.appendChild(button);
            buttons.set(id, button);
        }
        function removeButton(x, y) {
            const id = `button-${x}-${y}`; // get the unique ID of the button to remove
            const button = buttons.get(id); // get the button object from the Map
            if (button) {
                document.body.removeChild(button); // remove the button from the DOM
                buttons.delete(id); // remove the button from the Map
            }
        }
        function confirmMove(){
            addStone(lastX, lastY, playerTurn);
            playerTurn = (playerTurn % 2) + 1;
        }
        function addStone(x, y, turn){

            if (turn == 1){
                drawPiece(x,y, "#000");
                board[x][y] = 1;
            }
            else{
                drawPiece(x,y, "#FFF");
                board[x][y] = 2;
            }
            removeButton(x,y);

            const adjacentSpaces = [[x-1, y], [x+1, y], [x, y-1], [x, y+1], [x, y]];

            for (const space of adjacentSpaces){
                const [adjX, adjY] = space;
                let visited = [];
                console.log(adjX);
                if (board[adjX][adjY] != 0){
                    if (!findAdjacentFreeSpaces(adjX,adjY, visited)){
                        for (let t = 0; t < visited.length; t++){
                        console.log("visited: " + visited);
                        removeStone(visited[t][0], visited[t][1]);
                        }
                    }
                }
            }
        }
        function removeStone(x, y){
        addButton(x+1,y+1);
        board[x][y] = 0;

        removePiece(x,y);
        }
        function findAdjacentFreeSpaces(x, y, visited1) {
            // Add current position to visited list
            console.log("removing(" + x + ", " + y +")")

            // Check adjacent spaces
            const adjacentSpaces = [[x-1, y], [x+1, y], [x, y-1], [x, y+1]];
            if (x < 0 || y < 0 || x > 18 || y > 18) {
                return false; // Out of bounds, so return false
            }
            visited1.push([x, y]);
            let check = false;
            for (const space1 of adjacentSpaces) {
                const [adjX, adjY] = space1;
                if (board[adjX][adjY] === 0) {
                // This space is free
                return true;
                } else if (board[adjX][adjY] === board[x][y] && !visited1.some(pos => pos[0] === adjX && pos[1] === adjY)) {
                // This space has the same color stone and hasn't been visited yet
                console.log(visited1);
                if(findAdjacentFreeSpaces(adjX, adjY, visited1)){
                check = true;
                }
                }
            }
    visited = visited1;
  return check;
}

	</script>
</body>
</html>
