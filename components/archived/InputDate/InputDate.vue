<template>
    <div class="input-text input-text--date" :class="{'input-text--focused': focused}">
        <div v-if="label && label.length" class="input-text__label">
            <label :for="inputId">{{label}}</label>
        </div>
        <div class="input-text__wrapper">
            <div v-if="prepend" class="input-text__prepend">
                <slot name="prepend"/>
            </div>

            <input 
                tabindex="0"
                :id="inputId"
                v-model="inputValue"
                v-bind="$attrs"
                class="input-text__text"
                type="date"
                @focus="focus"
                @blur="blur">
            
            <div v-if="append" class="input-text__append">
                <slot name="append"/>
            </div>
            
            <div v-if="clear" class="input-text__clear">
                <button tabindex="0" v-if="inputValue" class="btn btn--icon" @click="inputValue = null">
                    <i class='uil uil-times'></i>
                </button>
            </div>

        </div>
    </div>
</template>

<script>
import InputTextBase from '@daniel-ordonez/do-input-text-base/InputTextBase'
export default {
    name: 'input-text',
    extends: InputTextBase,
    computed: {
        inputValue: {
            get () { return this.value ? this.value : this.content },
            set (value) { this.$emit('input', value); this.content = value }
        }
    }
}
</script>