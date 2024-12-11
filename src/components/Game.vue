//--------------------------------------------------------------
<template lang='pug'>
//----------------------- TEMPLATE -----------------------------
#outerWrapper
    #innerWrapper
        #game(v-if='numbersRemaining > 0')
            .number(
                v-for='(number, index) in numbers'
                :id='"number-" + index'
                @click='select($event, index)'
                :class='{"disabled": usedIndexes.includes(index), "hint": hintIndexes.includes(index)}'
                v-show='number != "X"'
            ) {{ number }}
        #win(v-else)
            span 🎊
            .title You Won
            span 🎉
        #controls
            #buttons
                .btn.action(@click='check()') Check
                .btn.action.disabled(@click='hint()') Hint
                .btn.action.warning(@click='restart()') Restart
            #score
                .section
                    span Total Numbers: {{ totalNumbers }}
                    span Total Lines: {{ totalLines }}
                .section
                    span Numbers Cleared: {{ numbersCleared }}
                    span Lines Cleared: {{ linesCleared }}
                .section
                    span Numbers Remaining: {{ numbersRemaining }}
                    span Lines Remaining: {{ linesRemaining }}
//--------------------------------------------------------------
</template>

<script>
// ----------------------- JAVASCRIPT --------------------------
export default {
    name: "GameView",
    data: () => {
        return {
            hintIndexes: [],
            linesCleared: 0,
            linesRemaining: 3,
            numbers: [],
            numbersCleared: 0,
            numbersRemaining: 27,
            selectedIndexes: [],
            startingNumbers: [1, 2, 3, 4, 5, 6, 7, 8, 9, 1, 1, 1, 2, 1, 3, 1, 4, 1, 5, 1, 6, 1, 7, 1, 8, 1, 9],
            totalLines: 3,
            totalNumbers: 0,
            usedIndexes: []
        }
    },

    // TODO load from cookies
    beforeMount() {
        // let board = cookies.get('board')
        // let indexes = cookies.get('usedIndexes')
        // if (board && indexes) {
        //     this.numbers = JSON.parse(board)
        //     this.usedIndexes = JSON.parse(indexes)
        //     this.linesCleared = cookies.get('linesCleared') || 0
        // } else this.numbers = [].concat(this.startingNumbers);
        // this.calculate()
    },

    async mounted() {
        this.numbers = this.startingNumbers
    },

    methods: {
        check() {
            let temp = [].concat(this.numbers)
            for (let i = 0; i < this.numbers.length; i++) {
                if (!this.usedIndexes.includes(i)) {
                    temp.push(this.numbers[i])
                }
            }
            this.numbers = temp
            this.calculate()
        },

        select(e, index) {
            if (!e.target.classList.contains('disabled')) {
                e.target.classList.add('selected')
                this.selectedIndexes.push(index)
                if (this.selectedIndexes.length == 2) {
                    let i1 = this.selectedIndexes[0]
                    let i2 = this.selectedIndexes[1]
                    if (this.isNeighbor()) {
                        if (this.isMatch(i1, i2)) {
                            this.disable(i1)
                            this.disable(i2)
                            this.numbersCleared += 2
                            this.numbersRemaining = this.numbers.length - this.numbersCleared
                        }
                    }
                    this.clearSelected()
                    this.clearLines()
                    this.save()
                }
            }
        },

        isNeighbor() {
            let i1 = this.selectedIndexes[0]
            let i2 = this.selectedIndexes[1]
            let max = Math.max(i1, i2)
            let min = Math.min(i1, i2)
            // horizontal neighbors
            let left = max - 1
            while (this.usedIndexes.includes(left) && left > 0) left--
            if (left == min) return true
            // vertical neighbors
            let up = max - 9
            while (this.usedIndexes.includes(up) && up > 9) up -= 9
            if (up == min) return true
        },

        isMatch(i1, i2) {
            let n1 = this.numbers[i1]
            let n2 = this.numbers[i2]
            return n1 == n2 || n1 + n2 == 10
        },

        disable(id) {
            let element1 = document.getElementById(`number-${id}`)
            if (element1) {
                element1.classList.remove('hint')
                element1.classList.add('disabled')
            }
            this.usedIndexes.push(id)
        },

        clearSelected() {
            for (let i = 0; i <= this.selectedIndexes.length; i++) {
                let element = document.getElementById(`number-${this.selectedIndexes[i]}`)
                if (element) element.classList.remove('selected')
            }
            this.selectedIndexes = []
        },

        hint() {
            
        },

        clearLines() {
            let lines = Math.ceil(this.numbers.length / 9)
            for (let l = 0; l < lines; l++) {
                let start = l * 9
                if (this.numbers[start] == 'X') continue
                if (
                    this.usedIndexes.includes(start) &&
                    this.usedIndexes.includes(start + 1) &&
                    this.usedIndexes.includes(start + 2) &&
                    this.usedIndexes.includes(start + 3) &&
                    this.usedIndexes.includes(start + 4) &&
                    this.usedIndexes.includes(start + 5) &&
                    this.usedIndexes.includes(start + 6) &&
                    this.usedIndexes.includes(start + 7) &&
                    this.usedIndexes.includes(start + 8)
                ) {
                    for (let i = start; i <= start + 8; i++) {
                        this.numbers[i] = 'X'
                    }
                    this.linesCleared++
                    this.calculate()
                }
            }
        },

        calculate() {
            this.totalLines = Math.ceil(this.numbers.length / 9)
            this.totalNumbers = this.numbers.length
            this.numbersCleared = this.usedIndexes.length
            this.numbersRemaining = this.totalNumbers - this.numbersCleared
            this.linesRemaining = (this.numbersRemaining == 0) ? 0 : this.totalLines - this.linesCleared
        },

        restart() {
            for (let i = 0; i <= this.usedIndexes.length; i++) {
                let element = document.getElementById(`number-${this.usedIndexes[i]}`)
                if (element) element.classList.remove('disabled')
            }
            this.clearSelected()
            this.hintIndexes = []
            this.numbers = [].concat(this.startingNumbers);
            this.usedIndexes = []
            this.calculate()
            this.save()
        },

        save() {
            // cookies.set('board', JSON.stringify(this.numbers))
            // cookies.set('usedIndexes', JSON.stringify(this.usedIndexes))
            // cookies.set('linesCleared', this.linesCleared)
        }
    }
}
//--------------------------------------------------------------
</script>
<style scoped>
/* ---------------------- STYLE ----------------------------- */
#innerWrapper {
    display: grid;
    justify-content: center;
    column-gap: 50px;
    background-color: white;
    padding: 50px;
    margin-bottom: 100px;
}
#game {
    display: grid;
    grid-template-columns: repeat(9, 1fr);
}
#win {
    font-size: 60px;
    font-weight: 900;
    display: flex;
}
#win .title {
    background: -webkit-linear-gradient(rgb(188, 12, 241), rgb(212, 4, 4));
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    margin: 0 20px;
}
.number {
    border: 1px solid #aaa;
    width: 50px;
    height: 50px;
    display: flex;
    justify-content: center;
    align-items: center;
}
.number.hint {
    background: #e2f9e2;
}
.number:not(.disabled):hover {
    cursor: pointer;
    background: #eee;
}
.number.selected {
    background: rgb(205, 230, 238) !important;
}
.number.disabled {
    background: #ddd;
    color: #cdcdcd;
}
#controls {
    position: fixed;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: white;
    display: grid;
    justify-content: center;
    gap: 20px;
    padding: 20px;
    box-shadow: 2px 2px 5px rgba(0, 0, 0, 0.2);
}
#score {
    display: flex;
    justify-content: center;
    gap: 50px;
}
#score .section {
    display: flex;
    gap: 15px;
}
#buttons {
    display: flex;
    justify-content: space-evenly;
}
.btn {
    background-color: rgb(3, 3, 197);
    color: white;
    padding: 5px 10px;
    font-weight: bold;
    border-radius: 8px;
    cursor: pointer;
}
.btn.disabled {
    background-color: lightgray;
}
.btn.warning {
    background-color: rgb(185, 0, 0);
}
</style>