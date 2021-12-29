<template>
    <div>
        <button @click="goToPage('home')">
            Go to home
        </button>
        <h1>
            Ranking
        </h1>
        <h2>These are the top voted songs: </h2>
        
            <table>
                    <tr>
                        <th>Ranking</th>
                        <th>Artist</th>
                        <th>Song</th>
                        <th>Points</th>
                    </tr>    
                    
                    <tr v-for="(song, index) in sortTheSongs(songs)" :key="index">
                        <td>
                           {{index+1}} 
                        </td>
                        <td>
                            {{ song.artist}}
                        </td>
                        <td>
                            {{ song.title }}
                        </td>
                        <td>
                            {{ song.points }}
                        </td>
                    </tr>
            </table>
            <h2>All votes in a wordcloud: </h2>
<vue-word-cloud
  font-size-ratio=0.09
  style="
    height: 480px;
    width: 640px;
  "
  :words="verdeling"
  :color="([, weight]) => weight > 1000 ? 'Lime' : weight > 500 ? 'mediumspringgreen' : weight > 100 ? 'seagreen' : 'darkgreen'"
  font-family="Roboto"
  animation-duration=4000
  animation-overlap=5
  
/>
<!-- <vue-bar-graph
  :points="verdeling"
  :width="1000"
  :height="300"
/>  -->      
    </div>
</template>




<script>
import VueBarGraph from 'vue-bar-graph';
import Vue from 'vue';
import VueWordCloud from 'vuewordcloud';


    export default {
        name: "Rankingpage",
        data() {
            return {
                songs: [],
                verdeling: [["","Points"]]
                };
        },
        mounted() {
            this.fetchSongs();
        },
        components:{
            VueBarGraph,
            [VueWordCloud.name]: VueWordCloud
        },
        methods: {
            // navigation method
            goToPage(page) {
                this.$emit("change-page", page);
            },
            // data methods
            fetchSongs() {
                // Get all songs
                const url = "http://webservies.be/eurosong/Songs";

                fetch(url)
                    .then((response) => {
                        return response.json();
                    })
                   .then((songs) => {
                        this.fetchArtists(songs);
                        songs.map((song)=> {
                            song.points = 0;
                    return song;
                })
                this.songs = songs;
                this.fetchVotes(songs);
                   });
            },

            fetchArtists(songs) {
                const url = "http://webservies.be/eurosong/Artists";
                fetch(url)
                    .then((response) => {
                        return response.json();
                    })
                    .then((artists) => {
                        songs.map((song) => {
                            const artist = artists.find((artist) => artist.id == song.artist);

                            song.artist = artist.name;
                            //song.vote = 0;
                            return song;
                        });
                        this.songs = songs;
                    });
                    /*.then(() => {
                        this.songs.map((song) => {
                            let songId = song.id;
                            const url = "http://webservies.be/eurosong/Songs/"+songId+"/points";
                            fetch(url)
                            .then((response) => {                                
                                return response.json();
                            })
                            .then((stemmen) => {
                                song.vote = stemmen;
                                console.log(song.vote);                                
                                return song;                            
                            })
                        });
                    });*/
                        
            },
            fetchVotes(songs){
            songs.map((song) => {
                const url = "http://webservies.be/eurosong/Songs/"+ song.id + "/points";
                
                fetch(url)
                .then((response) =>{
                    return response.json();
                })
                .then((points)=>{
                    song.points= points;

                    //wordcloud data aanmaken
                    var songdata = [song.title, song.points];
                    this.verdeling.push(songdata);
                })
                return song;
            })
            // change data of songs, so everything will get rerenderd
            this.songs = songs;        
        }, 
            sortTheSongs(songArr) { 
            return songArr.slice().sort(function(a, b) {
            return b.points - a.points;
            })
        }
    }   
}
</script>