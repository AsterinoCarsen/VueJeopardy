<template>
    <WagerModal 
        v-if="showWagerModal"
        :defaultWager="modalDefaultWager"
        :maxWager="modalMaxWager"
        @confirm="onWagerConfirmed"
    />

    <form class="game-selector" v-if="gameSettingsSelected === false" @submit.prevent="handleSettingsSubmit">
        <label>Number of Players</label>
        <select v-model.number="selectedNumPlayers">
            <option value="2">2</option>
            <option value="3">3</option>
            <option value="4">4</option>
            <option value="5">5</option>
            <option value="6">6</option>
        </select>

        <label>Number of Categories</label>
        <select v-model.number="selectedNumCategories">
            <option value="2">2</option>
            <option value="3">3</option>
            <option value="4">4</option>
            <option value="5">5</option>
            <option value="6">6</option>
        </select>

        <button type="submit">Submit</button>
    </form>
    <div v-if="gameSettingsSelected === true" class="parent-container">
        <div class="player-balances">
            <h3 v-for="(balance, i) in playersBalances" :key="i">Player #{{ i + 1 }} ${{ balance }}</h3>
        </div>
        <div class="header-container">
            <HeaderCard
                v-for="category in categories"
                :categoryTitle="category.name"
            />
        </div>

        <div class="grid-container" :style="`grid-template-columns: repeat(${selectedNumCategories}, 1fr)`">
            <DollarButtonCard
                v-for="(amount, i) in dollarAmounts"
                :key="i"
                :dollarAmount="amount"
                :categoryID="categories[Math.floor(i / 5)].id"
                :index="i"
                :ref="'card_' + i"
                @fetchedSuccessfully="disableButtonsAPITimeout"
                @playerAnswered="handlePlayerAnswer"
            />
        </div>

        <h1 class="won-display" v-if="this.gameFinished">Game Ended! Player # {{ playerWon }} won!</h1>
    </div>
</template>

<script>
import HeaderCard from './components/HeaderCard.vue';
import DollarButtonCard from './components/DollarButtonCard.vue';
import WagerModal from './components/WagerModal.vue';

export default {
    components: {
        HeaderCard,
        DollarButtonCard,
        WagerModal
    },
    data() {
        return {
            categories: ['Category1', 'Category2', 'Category3', 'Category4'],
            dollarAmounts: [
                "100", "200", "300", "400", "500",
                "100", "200", "300", "400", "500",
                "100", "200", "300", "400", "500",
                "100", "200", "300", "400", "500",
            ],
            currentActivePlayer: 1,
            playersBalances: [0, 0, 0],
            totalAnswered: 0,
            gameFinished: false,
            playerWon: -1,
            gameSettingsSelected: false,
            selectedNumPlayers: 3,
            selectedNumCategories: 4,
            pendingWagers: {},
            showWagerModal: false,
            modalCardIndex: null,
            modalDefaultWager: 0,
            modalMaxWager: 0,
        }
    },
    methods: {
        onWagerConfirmed(chosenWager) {
            this.pendingWagers[this.modalCardIndex] = chosenWager;
            this.showWagerModal = false;
            this.$refs['card_' + this.modalCardIndex][0].fetchQuestion(false);
        },
        handleSettingsSubmit() {
            this.playersBalances = Array(this.selectedNumPlayers).fill(0);
            const valuesPerCategory = ["100", "200", "300", "400", "500"];
            this.dollarAmounts = [];

            for (let i = 0; i < this.selectedNumCategories; i++) {
                this.dollarAmounts.push(...valuesPerCategory);
            }

            fetch("https://opentdb.com/api_category.php")
                .then(response => response.json())
                .then(data => {
                const allCategories = data.trivia_categories;

                const randomize = allCategories.sort(() => 0.5 - Math.random());

                this.categories = randomize.slice(0, this.selectedNumCategories);
                this.gameSettingsSelected = true;
            }).catch(error => { console.error('Error fetching mount api call: ', error)});
        },
        disableButtonsAPITimeout() {
            let buttons = document.querySelectorAll('button:not(.disabled-card)');
            buttons.forEach(btn => btn.disabled = true);

            setTimeout(() => {
                buttons = document.querySelectorAll('button:not(.disabled-card)');
                buttons.forEach(btn => btn.disabled = false);
            }, 5050);
        },
        handlePlayerAnswer({ index, gotCorrect, dollarAmount, isDoubleJeopardy }) {
            const specificCard = this.$refs['card_' + index][0];
            specificCard.passPlayerNumber(this.currentActivePlayer);

            let wager = dollarAmount;

            if (isDoubleJeopardy) {
                const bal = this.playersBalances[this.currentActivePlayer - 1];
                const face = dollarAmount;

                this.modalDefaultWager = face;
                this.modalMaxWager = bal < face ? face : bal;
                this.modalCardIndex = index;
                this.showWagerModal = true;
                return;
            }

            const amountToApply = this.pendingWagers[index] != null ? this.pendingWagers[index] : dollarAmount;

            if (gotCorrect) {
                this.playersBalances[this.currentActivePlayer - 1] += amountToApply;
            } else {
                this.playersBalances[this.currentActivePlayer - 1] -= amountToApply
            }

            delete this.pendingWagers[index];

            this.currentActivePlayer = this.currentActivePlayer % (this.selectedNumPlayers) + 1;
            this.totalAnswered++;

            if (this.totalAnswered === this.dollarAmounts.length) {
                this.gameFinished = true;

                let wonPlayerI = 0;
                for (let i = 0; i < this.playersBalances.length; i++) {
                    if (this.playersBalances[i] > this.playersBalances[wonPlayerI]) {
                        wonPlayerI = i;
                    }
                }

                this.playerWon = wonPlayerI + 1;
            }
        }
    }
}
</script>

<style scoped>
.parent-container {
    display: flex;
    flex-direction: column;
    width: 100%;
    height: 100%;
}

.header-container {
    display: flex;
}

.grid-container {
    display: grid;
    grid-template-rows: repeat(5, 1fr);
    grid-auto-flow: column;
    justify-items: center;
}

.player-balances {
    display: flex;
    width: 100%;
    justify-content: space-between;
    align-items: center;
    color: white;
}

.won-display {
    color: green;
}

.game-selector {
    display: flex;
    flex-direction: column;
    width: 25%;
    color: white;
    gap: 10px;
    background-color: rgb(65, 65, 65);
    padding: 10px;
    justify-self: center;
}
</style>
