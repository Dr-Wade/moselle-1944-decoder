<template>
    <div class="chiffrement">
        <textarea class="textarea" v-model="text" rows="10" cols="40"></textarea>
        <button class="process-button" :style="{ opacity: text ? '1' : '0.5' }" @click="encode">Encode</button>
        <div class="result" style="padding: 20px 5px">
            {{result}}
        </div>
    </div>
</template>
<script>
import { ASSOCATION_TABLE } from '../data'
export default {
    data: () => ({
        text: '',
        result: ''
    }),
    methods: {
        encode() {
            const letters = this.text.split('')
            this.result = letters.map((l) => {
                let i = Object.keys(ASSOCATION_TABLE).indexOf(l.toUpperCase())
                if (i > -1) return ASSOCATION_TABLE[l.toUpperCase()]
                else return l
            }).join(' ')
        }
    }
}
</script>
<style>
.chiffrement {
    width: 100%;
}
textarea {
    width: 100%;
    height: 30vh;
    border: dotted 1px #777777;
    border-radius: 20px;
}