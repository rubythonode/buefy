<template>
    <p
        class="control"
        :class="{ 'is-expanded': expanded }">
        <slot></slot>
        <span
            class="select"
            :class="[size, statusType, {
                'is-fullwidth': expanded,
                'is-loading': loading,
                'is-empty': selected === ''
            }]">

            <select
                v-model="selected"
                ref="select"
                :disabled="disabled"
                :readonly="readonly"
                :name="name"
                :required="required"
                @focus="$emit('focus', $event)"
                @blur="blur">

                <option
                    v-if="placeholder"
                    value=""
                    selected
                    disabled
                    hidden>
                    {{ placeholder }}
                </option>
                <template v-for="(option, i) in options">
                    <optgroup
                        v-if="isOptgroup(option, options[i - 1], i)"
                        :label="option.group">
                    </optgroup>
                    <option
                        :value="option.value"
                        :disabled="option.disabled">
                        {{ option.label }}
                    </option>
                </template>

            </select>
        </span>
    </p>
</template>

<script>
    export default {
        name: 'bSelect',
        props: {
            value: [String, Number, Object],
            size: String,
            placeholder: String,
            expanded: Boolean,
            loading: Boolean,

            // Native options to use in HTML5 validation
            name: String,
            disabled: Boolean,
            readonly: Boolean,
            required: Boolean
        },
        data() {
            return {
                options: [],
                selected: this.value || '',
                isValid: true,
                isSelectComponent: true // Used internally by Option
            }
        },
        computed: {
            /**
             * Type prop from parent if it's a Field.
             */
            statusType() {
                if (this.$parent.isFieldComponent) {
                    return this.$parent.newType
                }
            }
        },
        watch: {
            /**
             * When v-model is changed:
             *   1. Set the selected option.
             *   2. If it's invalid, validate again.
             */
            value(value) {
                this.selected = value || ''
                !this.isValid && this.html5Validation()
            },

            /**
             * When selected:
             *   1. Emit input event to update the user v-model.
             *   3. If it's invalid, validate again.
             */
            selected(value) {
                this.$emit('input', value)
                this.$emit('change', value)
                !this.isValid && this.html5Validation()
            }
        },
        methods: {
            /**
             * Verify if next item is a optgroup (another group chunk).
             */
            isOptgroup(option, previousOption, i) {
                if (!option.group) return
                // If it's first and has group property already show as subheader
                if (i === 0) return true

                if (previousOption === undefined) return
                return option.group !== previousOption.group
            },

            /**
             * Blur listener.
             * Fire the HTML5 validation.
             */
            blur(event) {
                this.$emit('blur', event)
                this.html5Validation()
            },

            /**
             * HTML5 validation, set isValid property.
             * If validation fail, send 'is-danger' type,
             * and error message to parent if it's a Field.
             */
            html5Validation() {
                let type = null
                let message = null
                let isValid = true
                if (!this.$refs.select.checkValidity()) {
                    type = 'is-danger'
                    message = this.$refs.select.validationMessage
                    isValid = false
                }
                this.isValid = isValid
                if (this.$parent.isFieldComponent) {
                    // Set type only if user haven't defined
                    if (!this.$parent.type) {
                        this.$parent.newType = type
                    }
                    // Set message only if user haven't defined
                    if (!this.$parent.message) {
                        this.$parent.newMessage = message
                    }
                }
            }
        }
    }
</script>
