<template>
  <div>
    <body>
    <div id="header">
      <h1>Set List Buddy</h1>
    </div>
      <div id="main">
        <div class="leftSide">
          <textarea name="rawList"
            class="textbox"
            cols="50" 
            rows="33" 
            placeholder="Paste song list here"
            v-model="state.rawList">
          </textarea>
          <h3 class="execute" @click="removeArtist">
          Remove Artists
          </h3>
        </div>

        <div class="rightSide">
          <div id="song-box">
            <textarea :class="{ gray : state.songArray.length == 0, white : state.songArray.length > 0 }"
                      class="textbox"        
                      cols="45" 
                      rows="26" 
                      v-model="songList">
            </textarea>
            <!-- <div v-for="song in state.songArray" v-bind:key="song.name">
              {{ song.name + song.key + song.request }}
            </div> -->
          </div>
        </div>
      </div>
    </body>
  </div>
 
  
</template>

<script>
import { reactive, computed } from 'vue';
export default {
  name: 'HomeView',
  components: {
    // HelloWorld
  },
  setup() {
    let state = reactive({
      rawList: '',
      songArray: [],
      keys: [ 'A', 'B' , 'C', 'D', 'E', 'F', 'G'],
      suffixes: ['#', 'b', 'm']
    });

  //computed
  let songList = computed(() => {
    let output = '';
    if (state.songArray.length > 0) {
      state.songArray.forEach(song => 
      output = output.concat(song.name + song.key + song.request + '\n'));
    }
    return output;
  });

//methods
    async function removeArtist() {
      state.songArray = await splitLines(state.rawList);
    }

    async function splitLines(rawList) { 
      let songArray = rawList.split(/\r\n|\r|\n/).filter(song => song != ''); 
      let objArray = [];
      for (var i = 0; i < songArray.length; i++) {
        //let obj = {};
        let obj = await findRequest(songArray[i]);
        obj.key = await findKey(songArray[i]);
        if (obj.key != '') {
           obj.key = ' (' + obj.key + ')';
        }
        obj.name = obj.name.substring(0, obj.name.indexOf('-')).trim();
        objArray.push(obj);
      }
      return objArray;
    }

    async function findRequest(song) {
      let obj = {};
      obj.request = '';
      while (song.substring(0, 1) == '*') {
        obj.request += '*';
        song = song.substring(1, song.length);
      }
      while (song.substring(song.length - 1, song.length) == '*') {
        obj.request += '*';
        song = song.substring(0, song.length - 1);
      }
      obj.name = song;
      return obj;
    }

    async function findKey(song) {
      let key = '';
      state.keys.forEach(k => {
      if (song.indexOf('(') > -1) {
        
          if (song.substring(song.indexOf('(') + 1, song.length).includes(k)) {
            key = k;
            const keyIndex = song.lastIndexOf(k);
            if (song.length > song.substring(keyIndex + 1)) {
              state.suffixes.forEach(suffix => {
                if (song.substring(keyIndex, keyIndex + 1) == suffix) {
                  key += suffix;
                }
              });
            }
          }
        } else {
          if (song.substring(song.length - 1, song.length) == k) {
            key = k;
          } else {
            state.suffixes.forEach(suffix => {
              if (song.substring(song.length - 2, song.length) == k + suffix) {
                key = k + suffix;
              }
            });
          } 
        }
      });
      
      return key;
    }

    return {
      removeArtist,
      songList,
      state
    }
  }
  
}
</script>
<style>
body {
  background-image: url('@/assets/broken-cake2.jpg');
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-size: cover;
}

#header {
  color: rgba(255, 255, 0, 0.786);
  margin-bottom: 30px;
}

h1 {
  margin-top: 10px;
  padding-top: 0px;
  font-size: 40px;
}

#main {
  display: flex;
  justify-content: space-evenly;
  z-index: 1;
  position: static;
}
.leftSide {
  display:flex;
  flex-direction: column;
  align-items: center;
}
/* #song-box {
  border: 1px solid black;
  width: 350px;
  height: 500px;
  border-radius: 5px;
  text-align: left;
  padding: 5px;
  color: black;
} */
.textbox {
  border-radius: 5px;
  padding: 5px;
  border: none;
}
.leftSide textarea::placeholder {
  font-size: 70px;
  line-height: 1.25;
  text-align: center;
  padding-top: 100px;
  padding-left: 15px;
  color: rgb(185, 185, 185);
}

.rightSide textarea {
  font-size: 15px;
  line-height: 1.25;
  text-align: left;
  padding-top: 10px;
  padding-left: 15px;
  color: rgb(0, 0, 0);
  font-family: Arial;
}

.execute {
  border: 1px solid black;
  border-radius: 5px;
  background-color: rgb(216, 216, 177);
  width: 170px;
}

.gray {
  background-color: rgb(206, 206, 206);
}
.white {
  background-color: rgb(255, 255, 255);
}

.execute:hover
 {
  cursor: pointer;
 }</style>