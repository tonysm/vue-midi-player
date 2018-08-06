<template>
    <div class="hello">
        <h1>MIDI Player</h1>

        <button @click="stop" v-if="playing">Stop</button>

        <form @submit.prevent="play">
            <div class="form-group">
                <label >Song</label>
                <select v-model="song">
                    <option :value="option" v-for="option in songs" :key="option.name">{{ option.name }}</option>
                </select>
            </div>

            <div class="form-group">
                <label for="">Instrument</label>
                <select v-model="instrument">
                    <option :value="option" v-for="option in instruments" :key="option.name">{{ option.name }}</option>
                </select>
            </div>

            <button :disabled="playing || !song || !instrument">
                Play
            </button>
        </form>
    </div>
</template>

<script>
    import { Player } from 'midi-player-js';
    import Soundfont from 'soundfont-player';
    import axios from 'axios';

    export default {
        name: 'HelloWorld',
        data () {
            return {
                song: null,
                instrument: null,
                playing: false,
                songs: [
                    { name: 'Everlong - Foo Fighters', filename: 'Foo_Fighters_-_Everlong.mid' },
                    { name: "Jurassic Park Movie Theme", filename: "Movie_Themes_-_Jurassic_Park.mid" },
                    { name: "Claire De Lune - Claude Debussy", filename: "clairdelune.mid" },
                ],
                instruments: [
                    { name: 'Piano', filename: 'acoustic_grand_piano-mp3' },
                    { name: 'Accordion', filename: 'accordion-mp3' },
                    { name: 'Acoustic Guitar Nylon', filename: 'acoustic_guitar_nylon-mp3' },
                    { name: 'Acoustic Guitar Steel', filename: 'acoustic_guitar_steel-mp3' },
                ],
            };
        },
        methods: {
            stop() {
                this.playing = false;
            },
            play() {
                this.playing = true;

                const AudioContext = window.AudioContext || window.webkitAudioContext || false;
                const ac = new AudioContext || new webkitAudioContext;

                Soundfont.instrument(ac, `https://raw.githubusercontent.com/gleitz/midi-js-soundfonts/gh-pages/MusyngKite/${this.instrument.filename}.js`).then((instrument) => {
                    const player = new Player((event) => {
                        if (!this.playing) {
                            player.stop();
                        }

                        if (event.name === 'Note on' && event.velocity > 0) {
                            instrument.play(event.noteName, ac.currentTime, {
                                gain: event.velocity / 100,
                            });
                        }
                    });

                    axios.get(`/songs/${this.song.filename}`, {
                        responseType: 'arraybuffer'
                    }).then(({data: song}) => {
                        player.loadArrayBuffer(song);

                        // we have access to each track, and each track has events.
                        // console.log(player.tracks);
                        player.play();
                    });
                });
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
    h3 {
        margin: 40px 0 0;
    }
</style>
