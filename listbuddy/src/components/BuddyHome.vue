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
          <div class="controls">
            <div class="separator">
              <h3 class="sep-text">Separator</h3>
              <div class="small-btn-row">
                <h3 class="btn square" :class ="{ selected : state.separators.includes('-'), unselected : !state.separators.includes('-') }" @click="toggleSeparator('-')">-</h3>
                <h3 class="btn square" :class ="{ selected : state.separators.includes(','), unselected : !state.separators.includes(',')  }" @click="toggleSeparator(',')">,</h3>
                <h3 class="btn square" :class ="{ selected : state.separators.includes('by'), unselected : !state.separators.includes('by')  }" @click="toggleSeparator('by')">by</h3>
              </div>
            </div>
            <h3 id="execute" class="btn" @click="removeArtist">Remove Artists</h3>
          </div>
        </div>

        <div class="rightSide">
          <div id="song-box">
            <textarea :class="{ gray : state.songArray.length == 0, white : state.songArray.length > 0 }"
                      class="textbox"        
                      cols="45" 
                      rows="26" 
                      v-model="songList">
            </textarea>
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
      separators: ['-'],
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
    function toggleSeparator(sep) {
      console.log('toggle ' + sep);
      if (state.separators.includes(sep)) {
        state.separators.splice(state.separators.indexOf(sep), 1);
      } else {
        state.separators.push(sep);
      }
    }

    async function removeArtist() {
      state.songArray = await splitLines(state.rawList);
      document.getElementById("song-box").scrollIntoView();
    }

    async function splitLines(rawList) { 
      let songArray = rawList.split(/\r\n|\r|\n/).filter(song => song != ''); 
      let objArray = [];
      for (var i = 0; i < songArray.length; i++) {
        let obj = await findRequest(songArray[i]);
        obj.key = await findKey(songArray[i]);
        if (obj.key != '') {
           obj.key = ' (' + obj.key + ')';
        }
        state.separators.forEach(sep => { 
          let sepIndex = obj.name.indexOf(sep);
          if (sepIndex != -1) {
            obj.name = obj.name.substring(0, obj.name.indexOf(sep)).trim();
          }
        });
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
            if (song.length > keyIndex + 1) {
              state.suffixes.forEach(suffix => {
                if (song.substring(keyIndex + 1, keyIndex + 2) == suffix) {
                  key += suffix;
                    if (suffix != 'm' && song.substring(keyIndex + 2, keyIndex + 3) == 'm') {
                      key += 'm';
                    }
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
      state,
      toggleSeparator
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
  /* margin-bottom: 30px; */
  text-shadow: 2px 1px #19191970;
  height: 10vh;
  min-height: 50px;
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

.textbox {
  border-radius: 5px;
  padding: 5px;
  border: none;
  width: 40vw;
  height: 70vh;
  min-height: 250px;
  resize: none;
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
  /* padding-left: 15px; */
  color: rgb(0, 0, 0);
  font-family: Arial;
}

.selected {
  background-color: rgb(244, 244, 202);
  border: 1px solid black;
}

.unselected {
  background-color: rgb(216, 216, 177);
  border: none;
}

.controls {
  margin-top: 20px;
  display: flex;
  justify-content: space-evenly;
  width: 40vw;
  height: 80px;
  background-color: white;
  border-radius: 5px;
}

.separator {
  display: flex;
  flex-direction: column;
  margin-top: 0px;
  width: 50%;
  flex: 1 1 0;
}

.sep-text {
  margin-top: 10px;
  margin-bottom: 0px;
  height: 10px;
}

.small-btn-row {
  display: flex;
  justify-content: space-evenly;
  margin-top: 0px;
  padding-left: 5%;
  padding-right: 10%;
  width: 90%;
}

.btn {
  border-radius: 5px;
}

.square {
  width: 28px;
  height: 25px;
  padding-top: 2px;
}

#execute {
  padding-top: 10px;
  padding-bottom: 10px;
  background-color: rgba(142, 142, 85, 0.707);
  border: none;
  font-size: 20px;
  margin-right: 3%;
  flex: 2 1 0;
}

.gray {
  background-color: rgb(206, 206, 206);
  opacity: 70%;
}
.white {
  background-color: rgb(255, 255, 255);
}

.btn:hover
 {
  cursor: pointer;
  opacity: 70%;
 }
 
 @media only screen and (max-width: 620px) {
  #main {flex-direction: column;}
  .textbox {width: 80vw;}
  .controls {width: 80vw;}
  .rightSide {margin-top: 20px;}
 }

 </style>