<template>
    <div class="input-text input--select" :class="{'input-text--focused': focused}">
        <div v-if="label && label.length" class="input-text__label">
            <label :for="inputId">{{label}}</label>
        </div>
        <div class="input-text__wrapper">
            <div v-if="prepend" class="input-text__prepend">
                <slot name="prepend"/>
            </div>

            <input 
                ref="input"
                tabindex="0"
                :id="inputId"
                v-model="inputValue"
                v-bind="$attrs"
                class="input-text__text"
                type="text"
                @keydown="keymonitor"
                @mousedown="dataList = true"
                @focus="focus"
                @blur="blur">
            
            <div v-if="clear" class="input-text__clear">
                <button tabindex="0" v-if="inputValue" class="btn btn--icon" @click="inputValue = null">
                    <i class='uil uil-times'></i>
                </button>
            </div>
            
            <div v-if="append" class="input-text__append">
                <slot name="append"/>
            </div>

            <div class="input-text__datalist" v-if="dataList && suggestions.length">
                <data-list ref="dataList">
                    <list-item v-for="(option, index) in suggestions" 
                        :key="`item-${index}-${option}`" 
                        @click="selectValue(option)">
                        {{option}}
                    </list-item>
                </data-list>
            </div>

        </div>
    </div>
</template>

<script>
import throttle from 'lodash/throttle'
import InputTextBase from '@daniel-ordonez/do-input-text-base/InputTextBase'
import DataList from '@daniel-ordonez/do-list/DataList'
import ListItem from '@daniel-ordonez/do-list/ListItem'

export default {
    name: 'input-select',
    extends: InputTextBase,
    components: {DataList, ListItem},
    data: () => ({
        dataList: false,
        filteredSuggestions: []
    }),
    props: {
        options: {
            type: Array,
            default: () => []
        }
    },
    computed: {
        inputValue: {
            get () { 
                let {value} = this
                return  value ? value : this.content 
            },
            set (value) {
                this.$emit('input', value)
                this.content = value
            }
        },
        suggestions () { return this.filteredSuggestions }
    },
    watch: {
        value (value) {
            if (!value || value === '' || !value.length) this.content = ''
            this.updateSuggestions()
        },
        options () {
            this.updateSuggestions()
        }
    },
    mounted () {
        document.body.addEventListener('click', event => {
            let closest = event.target.closest(`#${this.inputId}`)
            if (!closest) this.dataList = false
        })
        this.updateSuggestions()
    },
    methods: {
        filterOptions (filter, cb) {
            let {options} = this
            let suggestions = options.filter(
                item => typeof item === 'string' && item.match(new RegExp(`.*${filter}.*`))
            )
            console.log(suggestions)
            cb && cb(suggestions)
        },
        updateSuggestions () {
            let {options} = this
            let filter = this.content
            let cb = suggestions => {this.filteredSuggestions = suggestions}
            options.length && throttle(this.filterOptions, 300)(filter, cb)
        },
        focus () {
            this.focused = true
            this.$emit('focus')
            this.dataList = true
        },
        selectValue (value) {
            this.inputValue = value
            this.dataList = false
        },
        keymonitor (keyEvent) {
            if (!keyEvent || !keyEvent.key) return
            const key = keyEvent.key
            if (key === 'Tab') {
                this.dataList = false
            } else if (!this.dataList && this.focused && key === 'ArrowDown') {
                this.dataList = true
                return
            } else if (this.dataList) {
                let dataList = this.$refs.dataList
                if (!dataList) return
                if (key === 'ArrowDown') {
                    dataList.selectNext()
                    dataList.selected && dataList.selected.elm.scrollIntoView()
                }
                else if (key === 'ArrowUp') {
                    dataList.selectPrev()
                    dataList.selected && dataList.selected.elm.scrollIntoView()
                }
                else if (key === 'Enter' && dataList.selected) {
                    let value = this.suggestions[dataList.index]
                    this.selectValue(value)
                    keyEvent.preventDefault()
                }
            } 
        }
    }
}
</script>

<style>
.input-text__datalist {
    position: absolute;
    left: calc(-1 * var(--input--border-size, 0));
    right: calc(-1 * var(--input--border-size, 0));
    top: 100%;
    max-height: 200px;
    background: var(--color-bg);
    overflow-y: auto;
    z-index: 100;
    border: var(--input--border-size) solid var(--input--border-color);
}

</style>
