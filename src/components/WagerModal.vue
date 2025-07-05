<template>
    <div class="modal-overlay">
        <div class="modal-panel">
            <h2>Double Jeopardy!</h2>
            <p>Enter your wager (max ${{ maxWager }}):</p>
            <input 
                type="number"
                v-model.number="wager"
                :min="0"
                :max="maxWager"
                :disabled="!canCustomWager"
            />

            <div class="modal-buttons">
                <button :disabled="!canCustomWager" @click="confirm">Confirm</button>
                <button @click="useDefault">Default ${{ defaultWager }}</button>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'WagerModal',
    props: {
        maxWager: {
            type: Number,
            required: true
        },
        defaultWager: {
            type: Number,
            required: true
        }
    },
    data() {
        return {
            wager: this.defaultWager
        }
    },
    computed: {
        canCustomWager() {
            return this.maxWager > this.defaultWager;
        }
    },
    methods: {
        confirm() {
            const clampedValue = Math.min(Math.max(this.wager, 0), this.maxWager);
            this.$emit('confirm', clampedValue);
        },
        useDefault() {
            this.$emit('confirm', this.defaultWager);
        }
    }
}
</script>

<style scoped>
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
}

.modal-panel {
    background: #222;
    color: white;
    padding: 1.5rem;
    border-radius: 0.5rem;
    width: 300px;
    text-align: center;
}

.modal-buttons {
    margin: 0.5rem;
    padding: 0.5rem 1rem;
}
</style>