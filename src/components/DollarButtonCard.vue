<template>
    <button @click="fetchQuestion(true)" class="container" :class="{ 'disabled-card': isAnswered }" :disabled="isAnswered">
        <div v-if="this.isAnswered === false">
            <h2><b>{{ "$" + dollarAmount }}</b></h2>
            <p>{{ this.question }}</p>
            <p>{{ this.jeopardyDisplay }}</p>
            <div v-if="this.question !== ''">
                <button class="selection-button" @click.stop="handleSubmit(true)">True</button>
                <button class="selection-button" @click.stop="handleSubmit(false)">False</button>
            </div>
        </div>

        <div v-if="this.isAnswered === true">
            <h2 class="correct-outcome" v-if="outcome === 'Correct!'">{{ outcome }}</h2>
            <h2 class="incorrect-outcome" v-if="outcome === 'Incorrect!'">{{ outcome }}</h2>
            <p v-if="this.outcome === 'Correct!'">+${{ dollarAmount }}</p>
            <p class="correct-outcome" v-if="outcome === 'Correct!'">P# {{ this.playerNum }}</p>
            <p class="incorrect-outcome" v-if="outcome === 'Incorrect!'">P# {{ this.playerNum }}</p>
        </div>
    </button>
</template>

<script>
// Emit an event to App.vue when the fetch proceeds successfully to disable all other buttons for 5 seconds
export default {
    name: 'DollarAmount',
    props: {
        dollarAmount: {
            type: String,
            required: true
        },
        categoryID: {
            type: Number,
            required: true
        },
        index: {
            type: Number,
            required: true
        }
    },
    data() {
        return {
            question: "",
            answer: null,
            isAnswered: false,
            outcome: "",
            playerNum: -1,
            jeopardyDisplay: ""
        }
    },
    methods: {
        passPlayerNumber(playerNum) {
            this.playerNum = playerNum
        },
        handleSubmit(choice) {
            this.isAnswered = true;

            const gotCorrect = choice === this.answer;
            this.outcome = gotCorrect ? 'Correct!' : 'Incorrect!';
            this.$emit('playerAnswered', {
                index: this.index,
                gotCorrect,
                dollarAmount: parseInt(this.dollarAmount)
            });
        },
        fetchQuestion(rollDouble) {
            if (rollDouble) {
                const isDoubleJeopardy = Math.random() < 0.1;

                if (isDoubleJeopardy) {
                    this.jeopardyDisplay = "Double Jeopardy!";
                    this.$emit('playerAnswered', {
                        index: this.index,
                        gotCorrect: null,
                        dollarAmount: parseInt(this.dollarAmount),
                        isDoubleJeopardy: true
                    })
                    return;
                }
            }

            let difficulty = "medium";
            switch (this.dollarAmount) {
                case "100":
                    difficulty = 'easy';
                    break;
                case "200":
                    difficulty = 'easy';
                    break;
                case "300":
                    difficulty = 'medium';
                    break;
                case "400":
                    difficulty = 'medium';
                    break;
                case "500":
                    difficulty = 'hard';
                    break;
            }

            fetch(`https://opentdb.com/api.php?amount=1&category=${this.categoryID}&difficulty=${difficulty}&type=boolean`)
                .then(response => response.json())
                .then(data => {
                    const selectedQuestion = data.results[0].question;
                    const removedQuotations = selectedQuestion.replace(/&quot;/g, '"');
                    this.question = removedQuotations.replace(/&#039;/g, "'");

                    
                    const booleanString = data.results[0].correct_answer.toLowerCase();
                    if (booleanString === 'true') {
                        this.answer = true;
                    } else if (booleanString === 'false') {
                        this.answer = false;
                    }

                    this.$emit('fetchedSuccessfully');
                })
            .catch(error => { console.error("Failed fetching question for category " + this.categoryName)});
        }
    }
}
</script>

<style scoped>
.container {
    display: flex;
    flex-direction: column;
    width: 200px;
    height: 200px;
    background-color: rgb(50, 50, 50);
    color: white;
    justify-content: center;
    align-items: center;
    border: 2px solid black;

    cursor: pointer;
    transition: 0.3s ease;
}

.container:hover {
    background-color: rgb(65, 65, 65);
}

.container:disabled {
    background-color: rgb(11, 11, 11);
    cursor: not-allowed;
}

h2 {
    text-align: center;
}

.correct-outcome {
    color: green;
}

.incorrect-outcome {
    color: red;
}

.selection-button {
    background-color: rgb(40, 40, 40);
    color: white;
    border: 1px solid black;
    width: 65px;
    height: 35px;

    cursor: pointer;
    transition: 0.3s ease;
}

.selection-button:hover {
    background-color: rgb(55, 55, 55);
}
</style>