<template>
    <section id="silentbox-group">
        <slot></slot>
        <silentbox-overlay :moving="moving" />
    </section>
</template>

<script>
    import overlay from './overlay.vue';

    export default {
        name: 'SilentboxGroup',
        props: {
            transitionDuration: { type: Number, default: 200 }
        },
        data() {
            return {
                overlayVisibility: false,
                embedUrl: '',
                items: {
                    total: 0,
                    position: 0,
                    list: []
                },
                autoplay: false,
                moving: false,
                description: ''
            }
        },
        components: {
            'silentbox-overlay': overlay
        },
        methods: {
            // Open next item in the queue
            nextItem() {
                this.moving = true
                let item = this.$children[this.items.position + 1].$options._componentTag;

                if (this.items.position !== (this.items.total - 1)) {
                    this.items.position++;
                } else {
                    this.items.position = 0;
                }

                this.embedUrl = this.$children[this.items.position].src;

                this.autoplay = (this.$children[this.items.position].autoplay !== undefined)
                    ? this.$children[this.items.position].autoplay : false;

                this.description = (this.$children[this.items.position].description !== undefined)
                    ? this.$children[this.items.position].description : false;

                setTimeout(() => { this.moving = false }, this.transitionDuration)
            },
            // Open previous item in the queue
            prevItem() {
                this.moving = true
                let item = this.$children[this.items.position].$options._componentTag;

                if (this.items.position !== 0) {
                    this.items.position--;
                } else {
                    this.items.position = this.items.total - 1;
                }

                this.embedUrl = this.$children[this.items.position].src;

                this.autoplay = (this.$children[this.items.position].autoplay !== undefined)
                    ? this.$children[this.items.position].autoplay : false;

                this.description = (this.$children[this.items.position].description !== undefined)
                    ? this.$children[this.items.position].description : false;

                setTimeout(() => { this.moving = false }, this.transitionDuration)
            }
        },
        created() {
            this.$on('closeSilentboxOverlay', () => {
                this.overlayVisibility = false;
                this.$emit('changedVisibility', this.overlayVisibility)
            });
            this.$on('openSilentboxOverlay', item => {
                this.embedUrl = item.url;
                this.items.position = item.position;
                this.overlayVisibility = true;
                this.$emit('changedVisibility', this.overlayVisibility)
                this.autoplay = item.autoplay;
                this.description = item.description;
            });

            window.addEventListener('keyup', (event) => {
                if (event.which === 27) {
                    this.overlayVisibility = false;
                    this.$emit('changedVisibility', this.overlayVisibility)
                }
                if (event.which === 39) {
                    this.nextItem();
                }
                if (event.which === 37) {
                    this.prevItem();
                }
            });
        },
        mounted() {
            // When our vue component is loaded, we need to get count of silentbox-items
            this.items.total = 0;

            for (let i = 0; i < this.$children.length; i++) {
                if (this.$children[i].$options._componentTag === 'silentbox-item') {
                    this.items.total++;
                }
            }
        }
    }
</script>
