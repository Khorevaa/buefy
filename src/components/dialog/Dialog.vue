<template>
    <transition :name="animation">
        <div class="dialog modal is-active" v-if="isActive">
            <div class="modal-background" @click="cancel"></div>
            <div class="modal-card animation-content">
                <header class="modal-card-head" v-if="title">
                    <p class="modal-card-title">{{ title }}</p>
                </header>

                <section class="modal-card-body" :class="{ 'is-titleless': !title, 'is-flex': hasIcon }">
                    <div class="media">
                        <div class="media-left" v-if="hasIcon">
                            <b-icon
                                :icon="icon ? icon : iconByType"
                                :pack="iconPack"
                                :type="type"
                                :both="!icon"
                                size="is-large">
                            </b-icon>
                        </div>
                        <div class="media-content">
                            <p v-html="message"></p>

                            <div v-if="hasInput" class="field">
                                <div class="control">
                                    <input v-model="prompt"
                                        class="input"
                                        ref="input"
                                        required
                                        :class="{ 'is-danger': validationMessage }"
                                        v-bind="inputAttrs"
                                        @keyup.enter="confirm">
                                </div>
                                <p class="help is-danger">{{ validationMessage }}</p>
                            </div>
                        </div>
                    </div>
                </section>

                <footer class="modal-card-foot">
                    <button v-if="canCancel" class="button is-light" ref="cancelButton" @click="cancel">
                        {{ cancelText }}
                    </button>
                    <button class="button" :class="type" ref="confirmButton"  @click="confirm">
                        {{ confirmText }}
                    </button>
                </footer>
            </div>
        </div>
    </transition>
</template>

<script>
    import Icon from '../icon'
    import config from '../../utils/config'
    import { removeElement } from '../../utils/helpers'

    export default {
        inheritAttrs: false,
        components: {
            [Icon.name]: Icon
        },
        props: {
            title: String,
            message: String,
            icon: String,
            iconPack: String,
            hasIcon: Boolean,
            type: {
                type: String,
                default: 'is-primary'
            },
            confirmText: {
                type: String,
                default: () => {
                    return config.defaultDialogConfirmText
                        ? config.defaultDialogConfirmText
                        : 'OK'
                }
            },
            cancelText: {
                type: String,
                default: () => {
                    return config.defaultDialogCancelText
                        ? config.defaultDialogCancelText
                        : 'Cancel'
                }
            },
            animation: {
                type: String,
                default: 'zoom-out'
            },
            canCancel: {
                type: Boolean,
                default: true
            },
            hasInput: Boolean, // Used internally to know if it's prompt
            inputAttrs: {
                type: Object,
                default: () => {}
            },
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
            const prompt = this.hasInput
                ? this.inputAttrs.value || ''
                : ''

            return {
                isActive: false,
                savedScrollTop: null,
                prompt,
                validationMessage: ''
            }
        },
        computed: {
            /**
             * Icon name (MDI) based on the type.
             */
            iconByType() {
                switch (this.type) {
                    case 'is-info':
                        return 'information'
                    case 'is-success':
                        return 'check-circle'
                    case 'is-warning':
                        return 'alert'
                    case 'is-danger':
                        return 'alert-circle'
                    default:
                        return null
                }
            }
        },
        watch: {
            /**
             * Remove scrollbar from background to avoid weird scroll behavior.
             * while modal is active.
             */
            isActive() {
                if (typeof window !== 'undefined') {
                    this.savedScrollTop = !this.savedScrollTop
                        ? document.documentElement.scrollTop
                        : this.savedScrollTop

                    document.body.classList.toggle('is-noscroll', this.isActive)

                    if (this.isActive) {
                        document.body.style.top = `-${this.savedScrollTop}px`
                    } else {
                        document.documentElement.scrollTop = this.savedScrollTop
                        document.body.style.top = null
                        this.savedScrollTop = null
                    }
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
                    if (!this.$refs.input.checkValidity()) {
                        this.validationMessage = this.$refs.input.validationMessage
                        this.$nextTick(() => this.$refs.input.select())
                        return
                    }
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
                    removeElement(this.$el)
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
            this.isActive = true

            this.$nextTick(() => {
                // Handle which element receives focus
                this.hasInput
                    ? this.$refs.input.focus()
                    : this.$refs.confirmButton.focus()
            })
        },
        beforeDestroy() {
            if (typeof window !== 'undefined') {
                document.removeEventListener('keyup', this.keyPress)
            }
        }
    }
</script>
