<template lang="pug">
    div
        .text-center.pt-10
            .text-body-1 Восстановите порядок букв, используя подсказку
            .font-weight-bold.mt-5(v-if='game.letters && !game.new')  {{game.hint}}
            v-btn(rounded v-if='game.new' :loading="loading" color="primary" @click='getWords()' x-large dark).mt-10
                | Начать игру
        draggable(v-if='game.letters && !game.new' v-model="game.letters" draggable=".v-card" ghost-class="elevation-5" @end='joinLetters()').row.mt-10.justify-center
            v-card( v-for='i, index in game.letters' :key="index" height='75' width='75').letter.font-weight-bold.cyan.lighten-5.d-flex.justify-center.align-center {{i}}
        .text-center.mt-5(v-if='game.letters && !game.new') {{game.time}}
        v-dialog(v-model='game.victory' persistent scrollable max-width="400")
            v-card
                v-card-title.headline
                    | Вы выиграли! Повторить?
                v-card-text(v-if='result.show')
                    v-data-table(:headers='result.header' :items="result.items" hide-default-footer)
                        template(#item.index="{ item }")
                            | {{ result.items.indexOf(item) + 1 }}
                v-card-actions
                    v-spacer
                    v-btn(color="green darken-1" text @click='getWords()' :loading="loading")
                        | Конечно!
                    v-btn(v-if='!result.show' color="yellow darken-2" text @click='result.show = true' :loading="loading")
                        | Я уже устал :(
                    v-btn(v-else color="red darken-2" text @click='game.new = true, game.victory = false, result.show = false' :loading="loading")
                        | Закончить

        
</template>
<script>
    import draggable from 'vuedraggable';
    import Axios from "axios";

    export default {
        name: "game",
        data: () => ({
            loading: false,
            game: {
                id: null,
                value: null,
                letters: null,
                hint: null,
                new: true,
                victory: false,
                time: null
            },
            result: {
                show: false,
                header: [
                    {
                        text: '№',
                        value: 'index',
                    },
                    {
                        text: 'слово',
                        value: 'value',
                    },
                    {
                        text: 'время',
                        value: 'time'
                    }
                ],
                items:[]
            },
        }),
        components : {
            draggable
        },
        methods: {
            getWords() {
                this.loading = true
                this.result.show = false
                const randId =  Math.floor(Math.random() * (16 - 1) + 1)

                Axios({
                    method: 'get',
                    url: `http://localhost:3000/words?data.id=${randId}`,
                }).then(response => {
                    if (response.data[0] && response.data[0].data.chosen === false){
                        this.game.time = null
                        this.game.id = response.data[0].data.id
                        this.game.value = response.data[0].data.value.toUpperCase()
                        this.game.hint = response.data[0].data.hint
                        this.game.letters = this.game.value.split('').sort(() => Math.random() - 0.5)
                        this.game.new = false
                        this.game.victory = false
                        this.loading = false
                        this.newTime()
                    } else {
                        this.getWords();
                    }
                }).catch(error => {
                    console.log(error)
                });
            },
            joinLetters() {
                if (this.game.letters.join('') === this.game.value) {
                    this.game.victory = true
                }
            },
            newTime(){
                const options = {
                    minute: '2-digit',
                    second: '2-digit'
                }
                let startTime = 0
                
                let time = setInterval(() => {
                    startTime = startTime + 1000
                    this.game.time = new Intl.DateTimeFormat('ru-RU', options).format(startTime) 
                    
                    if (this.game.victory === true){
                        clearInterval(time);
                        this.result.items.push({
                            value: this.game.value,
                            time: this.game.time
                        })
                    }
                }, 1000)

            },
        },
    }
</script>
<style lang="scss" scoped>
    .letter {
        margin: 1px;
        user-select: none;
        cursor: pointer;
    }
</style>