<template>
    <v-container fluid>
        <div class="grid-container">
            <div v-for="tile in damboard" :key="tile.id" :class="['grid-cell', tileColor(tile.id)]"
                @click="checkerActions(tile)">
                <div v-if="tile.hasChecker" class="checker" :class="{ selected: isSelectedChecker(tile) }"
                    :style="{ backgroundColor: tile.checkerColor }"></div>
            </div>
        </div>
        <div class="status-box">
            <h2>{{ this.currentPlayer.name + ' is aan de beurt' }} - {{ 'team ' + this.currentPlayer.team }}</h2>
            <h3>Scores: {{ 'Team ' + this.player_1.team + ' score is ' + this.player_1.score }} - {{ 'Team ' + this.player_2.team + ' score is ' + this.player_2.score }}</h3>
        </div>
    </v-container>
</template>
<script>
import Statistics from "@/components/Statistics.vue";

export default {
    components: { Statistics },
    data() {
        return {
            damboard: this.initDamBoard(),
            selectedChecker: null,
            gridSize: 10,
            maxScore: 20,
            currentPlayer: null,
        };
    },
    props: {
        player_1: {
            type: Object,
            required: true,
        },
        player_2: {
            type: Object,
            required: true,
        }
    },
    created() {
        this.currentPlayer = this.player_1;
    },
    methods: {
        //This category contains functions that are mostly used to initialize the game and dynamic styling.
        switchPlayer() {
            if (this.currentPlayer.score < 20) {
                this.currentPlayer = this.currentPlayer.id === this.player_1.id ? this.player_2 : this.player_1;
            } else {
                alert(this.currentPlayer.name + 'has won the game');
                //resetknop
            }
        },
        /**
         * Generate an array of 10*10 (tiles) objects.
         * Each tile contains an ID, hasChecker Property and checkerColor property
         */
        initDamBoard() {
            this.gridSize = 10
            return Array.from({ length: this.gridSize * this.gridSize }, (_, index) => ({
                id: index + 1,
                hasChecker: this.isTileWithChecker(index + 1),
                checkerColor: this.isTileWithChecker(index + 1) ? this.checkerColor(index + 1) : ""
            }))
        },
        /**
         * This method checks a tile has a checker
         * It will return true when tileId falls within the first or last 4 rows and when the tile has the color wood-brown
         * @param {*} tileId 
         */
        isTileWithChecker(tileId) {
            this.gridSize = 10
            const row = Math.ceil(tileId / this.gridSize)
            const isInFirstOrLastFourRows = row <= 4 || row > this.gridSize - 4;
            const isCheckerTile = this.tileColor(tileId) === 'wood-brown';

            return isInFirstOrLastFourRows && isCheckerTile;
        },
        /**
         * This method determines the color of the checkers based on which side of the board they are.
         * @param {*} tileId 
         */
        checkerColor(tileId) {
            if (Math.ceil(tileId / this.gridSize) <= 4) {
                return 'white'
            } else if (Math.ceil(tileId / this.gridSize) > this.gridSize - 4) {
                return 'black'
            }
        },
        /**
         * Return the tile color based of the tileId
         * Return 0 for an even number 'wood-brown' or 1 odd for a 'marble-white' tile
         * @param {*} tileId 
         */
        tileColor(tileId) {
            return (tileId + Math.floor((tileId - 1) / this.gridSize)) % 2 === 0 ? 'wood-brown' : 'marble-white';
        },

        /**
         * Helper method to check whether tile has the same checker as this.selectedChecker
         * @param {} tile 
         */
        isSelectedChecker(tile) {
            return this.selectedChecker && this.selectedChecker.id === tile.id;
        },
        //The below methods are methods used for gamelogic
        /**
         * Main method for handeling checkerActions which describes multiple scenario's in which the game can be played out
         * Route 1: Checker is able to be captured resulting in this being the only move till its performed
         * Route 2: Normal Directional Movement
         * 
         * Switches players when the action is completed and unselects this.selectedChecker.
         * 
         */
        checkerActions(tile) {
            const potentialCaptures = this.checkForAvailableCaptures();
            if (potentialCaptures.length > 0) {
                if (tile.hasChecker && tile.checkerColor === this.currentPlayer.team) {
                    const checkerCaptures = potentialCaptures.filter(capture => capture.fromTile === tile.id)
                    if (checkerCaptures.length > 0) {
                        this.selectedChecker = tile;
                    }
                } else if (this.selectedChecker) {
                    const captureMove = potentialCaptures.find(capture => capture.fromTile === this.selectedChecker.id && capture.toTile === tile.id)
                    if (captureMove) {
                        this.moveChecker(this.selectedChecker, this.damboard.find(t => t.id === tile.id))
                        this.switchPlayer();
                        this.selectedChecker = null;
                    }
                }
            }
            else {
                if (tile.hasChecker && !this.selectedChecker && tile.checkerColor === this.currentPlayer.team) {
                    this.selectedChecker = tile;
                } else if (this.selectedChecker && this.isValidMove(this.selectedChecker, tile)) {
                    this.moveChecker(this.selectedChecker, tile);
                    this.switchPlayer();
                    this.selectedChecker = null;
                } else {
                    this.selectedChecker = null;
                }
            }

        },
        /**
         * This method checks whether a capturePlay can be performed.
         * returns the potentialCaptures array
         */
        checkForAvailableCaptures() {
            const potentialCaptures = [];
            this.damboard.forEach(tile => {
                if (tile.hasChecker && tile.checkerColor === this.currentPlayer.team) {
                    const allowedMoves = [];
                    this.determineAllowedMoves(tile, allowedMoves);
                    allowedMoves.forEach(move => {
                        if (this.isCaptureMove(tile.id, move)) {
                            potentialCaptures.push({ fromTile: tile.id, toTile: move });
                        }
                    })
                }
            })
            return potentialCaptures;
        },
        /**
         * This function checks whether a move is valid based off fromTile and toTile
         * @param {*} fromTile 
         * @param {*} toTile 
         * 
         * returns true if the move is valid false if it is not valid
         */
        isValidMove(fromTile, toTile) {
            const allowedMoves = []
            this.determineAllowedMoves(fromTile, allowedMoves)
            const captureMoves = allowedMoves.filter(move => this.isCaptureMove(fromTile.id, move))
            if (captureMoves.length > 0) {
                console.log(captureMoves)
                return captureMoves.includes(toTile.id)
            }
            return allowedMoves.includes(toTile.id)
        },

        /**
         * This method determines the directional movements the checker is able to perform
         * If a checker is black it can only move up, if a checker is white it can only move downwards.
         * @param {*} fromTile 
         * @param {*} allowedMoves 
         */
        determineAllowedMoves(fromTile, allowedMoves) {
            const upperRight = fromTile.id - (this.gridSize - 1);
            const upperLeft = fromTile.id - (this.gridSize + 1);
            const lowerLeft = fromTile.id + (this.gridSize - 1);
            const lowerRight = fromTile.id + (this.gridSize + 1);

            if (fromTile.checkerColor === "black") {
                this.checkIfDirectionalMovementIsValid(fromTile, upperRight, -9, allowedMoves)
                this.checkIfDirectionalMovementIsValid(fromTile, upperLeft, -11, allowedMoves)
            } else if (fromTile.checkerColor === "white") {
                this.checkIfDirectionalMovementIsValid(fromTile, lowerRight, 11, allowedMoves);
                this.checkIfDirectionalMovementIsValid(fromTile, lowerLeft, 9, allowedMoves)
            }
        },
        /**
         * This method determines whether a checker can be removed from the board.
         * If the the checker can be captured than it will be removed and a score will be added to the currentPlayer
         * @param {*} fromTileId 
         * @param {*} toTileId 
         */
        removeChecker(fromTileId, toTileId) {
            const canBeCaptured = this.isCaptureMove(fromTileId, toTileId);
            if (canBeCaptured) {
                const capturedTileId = this.getCapturedTileId(fromTileId, toTileId);
                const capturedTile = this.damboard.find(tile => tile.id === capturedTileId);

                capturedTile.hasChecker = false;
                capturedTile.checkerColor = null;
                this.currentPlayer.score += 1
                console.log(this.currentPlayer.score)
            }
        },

        /**
         * This method implements the movement logic of a checker and checks whether that move is a capturemove
         * @param {*} fromTile 
         * @param {*} toTile 
         */
        moveChecker(fromTile, toTile) {
            toTile.hasChecker = true;
            toTile.checkerColor = fromTile.checkerColor;
            fromTile.hasChecker = false;
            fromTile.checkerColor = null;

            console.log(fromTile)
            console.log(toTile)

            if (this.isCaptureMove(fromTile.id, toTile.id)) {
                this.removeChecker(fromTile.id, toTile.id);
            }
        },
        /**
         * Helper method that returns a tile which contains a checker that can be captured
         * @param {*} fromTileId 
         * @param {*} toTileId 
         */
        getCapturedTileId(fromTileId, toTileId) {
            return Math.floor((fromTileId + toTileId) / 2);
        },
        /**
         * Helper method that determines whether a move is a capture move
         * @param {*} fromTileId 
         * @param {*} toTileId 
         */
        isCaptureMove(fromTileId, toTileId) {
            const moveDifference = Math.abs(fromTileId - toTileId);
            return moveDifference === 18 || moveDifference === 22
        },
        /**
         * Helper method that checks whether a tile has a checker
         * @param {*} tileId 
         */
        tileHasChecker(tileId) {
            const tile = this.damboard.find(tile => tile.id === tileId);
            return tile && tile.hasChecker;
        },
        /**
         * This method determines whether a checker can be moved inside the board.
         * Route 1: It checks whether a checker is on an cornerTile, if it is than the current checker cannot move to that location
         * Route 2: It checks whether a checker close to the current one is from the opposite team, if it is then determine whether there is space to move behind that
         * checker.
         * Route 3: It makes normal movement to an empty tile
         * 
         * @param {*} fromTile 
         * @param {*} toTile 
         * @param {*} jump 
         * @param {*} allowedMoves 
         */
        checkIfDirectionalMovementIsValid(fromTile, toTilePos, jump, allowedMoves) {
            const cornerTiles = [1, 10, 11, 20, 21, 30, 31, 40, 41, 50, 51, 60, 61, 70, 71, 80, 81, 90, 91, 100];

            if (cornerTiles.includes(toTilePos)) {
                const targetTile = this.damboard.find(tile => tile.id === toTilePos);
                if (targetTile && targetTile.hasChecker) {
                    return; 
                }
            }
            const toTile = this.damboard.find(tile => tile.id === toTilePos);
            if (toTile && toTile.hasChecker) {
                if (toTile.checkerColor !== fromTile.checkerColor) {
                    const landingTileId = toTilePos + jump;
                    const landingTile = this.damboard.find(tile => tile.id === landingTileId)
                    if (landingTile && !landingTile.hasChecker) {
                        allowedMoves.push(landingTileId)
                    }
                }
            } else {
                allowedMoves.push(toTilePos);
            }
        },
    }
}
</script>
<style>
.grid-container {
    display: grid;
    grid-template-columns: repeat(10, 1fr);
    grid-template-rows: repeat(10, 1fr);
    width: 500px;
    height: 500px;
    gap: 2px;
}

.grid-cell {
    width: 100%;
    height: 100%;
}

.marble-white {
    background-color: #f0f0f0;
}

.wood-brown {
    background-color: #8B4513;
}

.checker {
    width: 80%;
    height: 80%;
    border-radius: 50%;
    /* Color for the checker pieces */
    margin: auto
}

.white {
    background-color: aliceblue;
}

.black {
    background-color: black;
}

.selected {
    border: 2px solid blue;
}
</style>