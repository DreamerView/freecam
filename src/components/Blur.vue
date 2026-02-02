<template>
    <div class="blur" @click.self="$emit('play')">
        <div class="modal-block">
            <slot>
                <div style="display:flex;flex-direction: column; gap:2.5rem;">
                    <h1 style="text-align: center;">Инструкция</h1>
                    <div style="display:flex;gap:2rem;flex-wrap: wrap;">
                        <div style="display:flex;flex-direction:column;gap:1rem">
                            <div class="hint">
                                <kbd>W</kbd>
                                <span>Движение вперёд</span>
                            </div>

                            <div class="hint">
                                <kbd>A</kbd>
                                <span>Движение влево</span>
                            </div>

                            <div class="hint">
                                <kbd>S</kbd>
                                <span>Движение назад</span>
                            </div>

                            <div class="hint">
                                <kbd>D</kbd>
                                <span>Движение вправо</span>
                            </div>
                        </div>
                        <div style="display:flex;flex-direction:column;gap:1rem">
                            <div class="hint">
                                <kbd>ESC</kbd>
                                <span>В паузу</span>
                            </div>
                            <div class="hint">
                                <kbd>Space</kbd>
                                <span>Подъём камеры</span>
                            </div>

                            <div class="hint">
                                <kbd>Ctrl</kbd>
                                <span>Опускание камеры</span>
                            </div>
                            <div class="hint">
                                <kbd>🖱</kbd>
                                <span>Обзор — движение мыши</span>
                            </div>
                        </div>
                    </div>

                    <button @click="$emit('play')">
                        ▶ Продолжить
                    </button>
                </div>

            </slot>
        </div>
    </div>
</template>

<script setup>
    defineEmits(['play'])
</script>

<style scoped>
    kbd {
        background-color: #111;
        border-radius: 0.5rem;
        border: 1px solid #b4b4b4;
        box-shadow:
            0 1px 1px rgba(0, 0, 0, 0.2),
            0 2px 0 0 rgba(255, 255, 255, 0.7) inset;
        color: #fff;
        display: inline-block;
        font-size: 0.8rem;
        font-weight: 700;
        line-height: 1;
        padding: 8px;
        white-space: nowrap;
    }

    .hint {
        display: flex;
        align-items: center;
        gap: 0.75rem;

        font-size: 14px;
        line-height: 1.4;

        color: #111;
    }

    .blur {
        width: 100vw;
        height: 100dvh;
        background: rgba(255, 255, 255, 0.2);
        box-shadow: 0 4px 30px rgba(0, 0, 0, 0.1);
        backdrop-filter: blur(16px);
        -webkit-backdrop-filter: blur(16px);

        display: flex;
        align-items: center;
        justify-content: center;
    }

    .modal-block {
        margin:1rem;
        padding: 0.5rem 2rem;
        max-width: 450px;
        width: 100%;
        aspect-ratio: 1;
        background-color: rgba(255, 255, 255, 0.7);
        border-radius: 2rem;

        display: flex;
        align-items: center;
        justify-content: center;
    }

    button {
        padding: 0.8rem 1.5rem;
        font-size: 16px;
        border-radius: 999px;
        border: none;
        cursor: pointer;
        background-color: #0d6efd;
        color:#fff;
    }
</style>