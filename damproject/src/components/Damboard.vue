<template>
    <v-container fluid>
        <div class="grid-container">
            <div v-for="tile in damboard" :key="tile.id" :class="['grid-cell', tileColor(tile.id)]"
                @click="checkerActions(tile)">
                <div v-if="tile.hasChecker" class="checker" :style="{ backgroundColor: tile.checkerColor }"></div>
            </div>
        </div>
    </v-container>
</template>
<script>
export default {
    data() {
        return {
            damboard: this.initDamBoard(),
            selectedChecker: null,
            gridSize: 10,
            maxScore: 20,
            player_1: {
                id: 1,
                name: "Player 1",
                team: "white",
                score: 0,
            },
            player_2: {
                id: 2,
                name: "Player 2",
                team: "black",
                score: 0,
            },
            currentPlayer: null,
        };
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
            }
        },
        initDamBoard() {
            this.gridSize = 10
            return Array.from({ length: this.gridSize * this.gridSize }, (_, index) => ({
                id: index + 1,
                hasChecker: this.isTileWithChecker(index + 1),
                checkerColor: this.isTileWithChecker(index + 1) ? this.checkerColor(index + 1) : ""
            }))
        },
        isTileWithChecker(tileId) {
            this.gridSize = 10
            const row = Math.ceil(tileId / this.gridSize)
            const isInFirstOrLastFourRows = row <= 4 || row > this.gridSize - 4;
            const isCheckerCell = this.tileColor(tileId) === 'wood-brown';

            return isInFirstOrLastFourRows && isCheckerCell;
        },
        checkerColor(tileId) {
            if (Math.ceil(tileId / this.gridSize) <= 4) {
                return 'white'
            } else if (Math.ceil(tileId / this.gridSize) > this.gridSize - 4) {
                return 'black'
            }
        },
        tileColor(tileId) {
            return (tileId + Math.floor((tileId - 1) / this.gridSize)) % 2 === 0 ? 'wood-brown' : 'marble-white';
        },

        //The below methods are methods used for gamelogic
        checkerActions(tile) {
            const potentialCaptures = this.checkForAvailableCaptures();
            const maxScore = 20;
            // if (captureAvailable) {
            //     if (tile.hasChecker && tile.checkerColor === this.currentPlayer.team) {
            //         this.selectedChecker = tile;
            //         if (this.selectedChecker && this.isValidMove(this.selectedChecker, this.hitTile)) {
            //             this.moveChecker(this.selectedChecker, this.hitTile);
            //             this.switchPlayer();
            //             this.selectedChecker = null;
            //         }
            //     }
            // }
            if(potentialCaptures.length > 0){
                if(tile.hasChecker && tile.checkerColor === this.currentPlayer.team){
                    const checkerCaptures = potentialCaptures.filter(capture => capture.fromTile === tile.id)
                    if(checkerCaptures.length > 0){
                        this.selectedChecker = tile;
                    }
                } else if (this.selectedChecker){
                    const captureMove = potentialCaptures.find(capture => capture.fromTile === this.selectedChecker.id && capture.toTile === tile.id)
                    if(captureMove){
                        this.moveChecker(this.selectedChecker, this.damboard.find(t => t.id === tile.id))
                        this.switchPlayer();
                        this.selectedChecker = null;
                    }
                }
            }
            else {
                // Normal play without mandatory captures
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
        checkForAvailableCaptures() {
            // const playerCheckers = this.damboard.filter(tile => tile.checkerColor === this.currentPlayer.team && tile.hasChecker);
            // for (const checker of playerCheckers) {
            //     const allowedMoves = [];
            //     this.determineAllowedMoves(checker, allowedMoves);
            //     const captureMoves = allowedMoves.filter(move => this.isCaptureMove(checker.id, move));
            //     if (captureMoves.length > 0 && captureMoves.length === 1) {
            //         this.hitTile = this.damboard.find(tile => tile.id === captureMoves[0])
            //         return true; // Capture available
            //     } else if (captureMoves.length > 1) {
            //         this.hitTile
            //         return true;
            //     }
            // }
            // return false; // No captures available

            const potentialCaptures = [];
            this.damboard.forEach(tile => {
                if(tile.hasChecker && tile.checkerColor === this.currentPlayer.team){
                    const allowedMoves = [];
                    this.determineAllowedMoves(tile, allowedMoves);
                    allowedMoves.forEach(move => {
                        if(this.isCaptureMove(tile.id, move)){
                            potentialCaptures.push({fromTile: tile.id, toTile: move});
                        }
                    })
                }
            })
            return potentialCaptures;
        },
        /**
         * This function checks whether a move is valid based off
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

        determineAllowedMoves(fromTile, allowedMoves) {
            const upperRight = fromTile.id - (this.gridSize - 1);
            const upperLeft = fromTile.id - (this.gridSize + 1);
            const lowerLeft = fromTile.id + (this.gridSize - 1);
            const lowerRight = fromTile.id + (this.gridSize + 1);

            //conditions when the currently selected tile has a black checker
            if (fromTile.checkerColor === "black") {
                this.checkIfDirectionalMovementIsValid(fromTile, upperRight, -9, allowedMoves)
                this.checkIfDirectionalMovementIsValid(fromTile, upperLeft, -11, allowedMoves)
            } else if (fromTile.checkerColor === "white") {
                this.checkIfDirectionalMovementIsValid(fromTile, lowerRight, 11, allowedMoves);
                this.checkIfDirectionalMovementIsValid(fromTile, lowerLeft, 9, allowedMoves)
            }
        },

        removeChecker(fromTileId, toTileId) {
            // console.log("coordinates are", fromTileId, toTileId)
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

        getCapturedTileId(fromTileId, toTileId) {
            return Math.floor((fromTileId + toTileId) / 2);
        },

        isCaptureMove(fromTileId, toTileId) {
            const moveDifference = Math.abs(fromTileId - toTileId);
            return moveDifference === 18 || moveDifference === 22
        },

        tileHasChecker(tileId) {
            const tile = this.damboard.find(tile => tile.id === tileId);
            return tile && tile.hasChecker;
        },

        checkIfDirectionalMovementIsValid(fromTile, toTilePos, jump, allowedMoves) {
            // Define the corner tiles
            const cornerTiles = [1, 10, 11, 20, 21, 30, 31, 40, 41, 50, 51, 60, 61, 70, 71, 80, 81, 90, 91, 100];
            // Check if the targetTile is a corner tile and occupied
            if (cornerTiles.includes(toTilePos)) {
                const targetTile = this.damboard.find(tile => tile.id === toTilePos);
                if (targetTile && targetTile.hasChecker) {
                    return; // Do not add to allowed moves if the corner tile is occupied
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
</style>