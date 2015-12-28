Changes
=================
This was changed by Bryan Kwon

1. Abstract
==================

This is a real-time tic tac toe game, in which two players can play against each other over a network, and unlimited users can watch the game. The game consist of a server (node.js application) and client (javascript application)

2. Technology
==================

The game is using long-polling connections for communication between the client and the server. Each received message by the server is broadcasted to all open connections. Websockets would be more reliable. Since an event can happen while some of the client are reconnecting, a message queue should be implemented, and clients should send the timestamp of the last message they received. This is not implemented in the current release.

3. Server
==================

The server responds to POST requests with json formatted data. Each message should provide room and command parameters, and may provide any other parameters used by the corresponding commands.

The client should send init as the first command. While no response is received, the client is the only connected to the room given and game can't be started. As soon as there is second player, the server responds to the init command, telling each client which player it is.

The set command is issued when some of the client performs a move; the provided additional data is send to all other connections, watching this room.

The wait command does not have any parameters; it just adds the connection to the pool, so when someone sends set the client will receive the data.

The reset command (not-implemented), indicating that a new game was started between the players, and all connected clients should reset their boards/

The server does not store the actual state, which means that any new watcher can't initialize itself with the current game state and will see only the new moves. Server however could store the state, and upon restart ask the first connected players for the current game state, and the client should send a data command, with all the game data, which will be broadcasted to the newly connected watchers or to a reconnected player (if the player accidentally closes the browser for example).

4. Client
==================

The client has 3 main components – the Board, Game and Connection classes.

The Connection class encapsulates the connection to the server and has only one method – send, which accepts 3 parameters – the command to be issued, the data to be send along with the command, and callback to be called when there is response. The response is also a JSON object, containing both command name and any data associated with the command. If there is a connection error, the last command will be automatically resent, with increasing pause between retries (up to 10 seconds).

The Game class initializes the connection and the board and it enables/disables the board, depending on whether it's the  current user's turn to move or not. The Game class uses event listeners to interact with the board, and it display the status message (if it's the user's turn to move or not, if the game is over, the game result, etc).

The Board class is the core of the game. It accepts an HTML Table element as the only parameter of its constructor and binds the game board to the table cells. When user clicks on the cell, a board:set event is dispatched, with the row and cell number. If the move was winning, a board:winning event is dispatched, and if the last cell is filled and it's not a winning move – a board:full event is dispatched.
