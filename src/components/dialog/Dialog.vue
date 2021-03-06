<template>
    <transition
        appear
        appear-active-class="fadeIn"
        enter-active-class="fadeIn"
        leave-active-class="fadeOut">

        <div class="dialog modal is-active" v-if="isActive">
            <div class="modal-background" @click="cancel"></div>

            <transition
                appear
                appear-active-class="zoomIn"
                enter-active-class="zoomIn">

                <div class="modal-card" v-if="isActive">
                    <header class="modal-card-head" v-if="title">
                        <p class="modal-card-title">{{ title }}</p>
                    </header>
                    <section class="modal-card-body" :class="{ 'is-titleless': !title, 'is-flex': hasIcon }">
                        <b-icon
                            :icon="icon"
                            :class="type"
                            both
                            size="is-large custom-icon"
                            v-if="icon && hasIcon">
                        </b-icon>
                        <p v-html="message"></p>

                        <b-field v-if="hasInput">
                            <b-input
                                v-model="prompt"
                                ref="input"
                                :placeholder="inputPlaceholder"
                                required
                                :maxlength="inputMaxlength"
                                :name="inputName"
                                @keyup.enter.native="confirm">
                            </b-input>
                        </b-field>
                    </section>
                    <footer class="modal-card-foot">
                        <button class="button is-light" ref="cancelButton" @click="cancel" v-if="canCancel">{{ cancelText }}</button>
                        <button class="button" ref="confirmButton" :class="type" @click="confirm">{{ confirmText }}</button>
                    </footer>
                </div>

            </transition>

        </div>

        </transition>
</template>

<script>
    import Icon from '../icon'
    import Input from '../input'

    export default {
        components: {
            [Icon.name]: Icon,
            [Input.name]: Input
        },
        props: {
            title: String,
            message: String,
            hasIcon: Boolean,
            type: {
                type: String,
                default: 'is-primary'
            },
            confirmText: {
                type: String,
                default: 'OK'
            },
            cancelText: {
                type: String,
                default: 'Cancel'
            },
            canCancel: {
                type: Boolean,
                default: true
            },
            hasInput: Boolean, // Used internally to know if it's prompt
            inputPlaceholder: String,
            inputName: String,
            inputMaxlength: [Number, String],
            onConfirm: {
                type: Function,
                default: () => {}
            },
            onCancel: {
                type: Function,
                default: () => {}
            }
        },
        data() {
            return {
                isActive: true,
                prompt: ''
            }
        },
        computed: {
            /**
             * Icon name (MDI) based on the type.
             */
            icon() {
                switch (this.type) {
                    case 'is-info':
                        return 'info'
                    case 'is-success':
                        return 'check_circle'
                    case 'is-warning':
                        return 'warning'
                    case 'is-danger':
                        return 'error'
                    default:
                        return null
                }
            }
        },
        methods: {
            /**
             * If it's a prompt Dialog, validate the input.
             * Call the onConfirm prop (function) and close the Dialog.
             */
            confirm() {
                if (this.$refs.input !== undefined) {
                    this.$refs.input.html5Validation()
                    if (!this.$refs.input.isValid) return
                }

                this.onConfirm(this.prompt)
                this.close()
            },

            /**
             * Call the onCancel prop (function) and close the Dialog.
             */
            cancel() {
                if (!this.canCancel) return

                this.onCancel()
                this.close()
            },

            /**
             * Close the Dialog.
             */
            close() {
                this.isActive = false

                // Timeout for the animation complete before destroying
                setTimeout(() => {
                    this.$destroy()
                    this.$el.remove()
                }, 150)
            },

            /**
             * Keypress event that is bound to the document.
             */
            keyPress(event) {
                // Esc key
                if (event.keyCode === 27) this.cancel()
            }
        },
        created() {
            if (typeof window !== 'undefined') {
                document.addEventListener('keyup', this.keyPress)
            }
        },
        beforeMount() {
            // Insert the Dialog component in body tag
            document.body.appendChild(this.$el)
        },
        mounted() {
            // Handle which element receives focus
            if (this.hasInput) {
                this.$refs.input.$refs.input.focus()
            } else if (this.canCancel) {
                this.$refs.cancelButton.focus()
            } else {
                this.$refs.confirmButton.focus()
            }
        },
        beforeDestroy() {
            if (typeof window !== 'undefined') {
                document.removeEventListener('keyup', this.keyPress)
            }
        }
    }
</script>
