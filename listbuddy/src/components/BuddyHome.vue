<template>
  <div>
    <body>
      <div id="header">
        <h1>Set List Buddy</h1>
      </div>
      <div id="main" :class="{ addRight : state.songArray.length > 0 }">
        <div class="leftSide" :class="{ left : state.songArray.length > 0 }">
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
                <h3 class="btn square" :class ="{ selected : state.separators.includes(' by '), unselected : !state.separators.includes(' by ')  }" @click="toggleSeparator(' by ')">by</h3>
              </div>
            </div>
            <div id="action-btns">
              <h3 id="execute" class="btn" @click="removeArtist">Remove Artists</h3>
              <h3 id="clear" class="btn" @click="state.rawList = ''">Clear Song List</h3>
            </div>
            
          </div>
        </div>
        <div id="results">
          <div v-if="state.songArray.length > 0" class="rightSide">
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
      suffixArrays: {
        '#' : ['#', 'sharp'],
        'b' : ['b', 'flat'],
        'm' : ['m']
      }
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
      if (state.separators.includes(sep)) {
        state.separators.splice(state.separators.indexOf(sep), 1);
      } else {
        state.separators.push(sep);
      }
    }

    async function removeArtist() {
      state.songArray = await splitLines(state.rawList);
      document.getElementById("results").scrollIntoView();
    }

    async function splitLines(rawList) { 
      let songArray = rawList.split(/\r\n|\r|\n/).filter(song => song != ''); 
      let objArray = [];
      for (var i = 0; i < songArray.length; i++) {
        let obj = await findRequest(songArray[i]);
        obj = await findName(obj);
        obj.key = obj.key ? await findKey(obj.key) : '';
        if (obj.key != '') {
           obj.key = ' (' + obj.key + ')';
        }
        objArray.push(obj);
      }
      return objArray;
    }

    async function findName(obj) {
      state.separators.forEach(sep => { 
        let sepIndex = obj.name.indexOf(sep);
        if (sepIndex != -1) {
          obj.key = obj.name.substring(obj.name.indexOf(sep) + 1, obj.name.length);
          obj.name = obj.name.substring(0, obj.name.indexOf(sep)).trim();
        }
      });
      return obj;
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

    async function findKey(str) {
      let songKey = '';
      state.keys.forEach(k => {
        if ((str.indexOf('(') > -1 && str.substring(str.indexOf('(') + 1, str.length).includes(k)) 
        || (str.substring(str.length - 2, str.length).includes(k))) {
          songKey = k;
          const keyIndex = str.lastIndexOf(k);
          songKey = getSuffix(str, keyIndex, songKey);
        } 
      });
      return songKey;
    }

    function getSuffix(str, keyIndex, songKey) {
      if (str.length > keyIndex + 1) {
        for (const [key, value] of Object.entries(state.suffixArrays)) {
          value.forEach(suffix => {
            if (!songKey.includes(suffix) && str.substring(keyIndex + 1, keyIndex + 2 + suffix.length).includes(suffix)) {
              songKey += key;
              if (suffix != 'm' && !songKey.includes('m') 
              && str.substring(keyIndex + 1 + suffix.length, keyIndex + 4 + suffix.length).includes('m')) {
                songKey += 'm';
              }
            }
          });
        }
      }
      return songKey;
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
  /* display: flex; */
  justify-content: space-evenly;
  z-index: 1;
  position: static;
}

.addRight {
  display: flex;
}

@keyframes move-left {
  from {margin-left: 28vw;}
  to {margin-left: 0vw;
    align-items: center;
  }
}

.left {
  margin-left: 28vw;
  animation-name: move-left;
  animation-duration: 1s;
  animation-fill-mode: forwards;
}

.leftSide {
  display:flex;
  flex-direction: column;
  width: 40vw;
  align-items: center;
  margin-left: 28vw;
}

.textbox {
  border-radius: 5px;
  padding: .5vw;
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
  color: rgb(0, 0, 0);
  font-family: Arial;
  animation: fadeIn 1s;
  -webkit-animation: fadeIn 1s;
  -moz-animation: fadeIn 1s;
  -o-animation: fadeIn 1s;
  -ms-animation: fadeIn 1s;
}

@keyframes fadeIn {
  0% { opacity: 0; }
  10% { opacity: 0; }
  100% { opacity: 1; }
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
  width: 41vw;
  height: 81px;
  background-color: white;
  border-radius: 5px;
  align-self: center;
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

#action-btns {
  flex: 2 1 0;
  display: flex;
  flex-direction: column;
  height: 5vh;
  margin-right: 3%;
}

#execute {
  padding-top: 7px;
  padding-bottom: 7px;
  background-color: rgba(142, 142, 85, 0.707);
  border: none;
  font-size: 20px;
  margin-top: 5px;
  margin-bottom: 6px;
}

#clear {
  margin-top: 0px;
  align-self: flex-start;
  background-color: rgb(216, 216, 177);
  border: none;
  padding-top: 3px;
  padding-bottom: 3px;
  padding-left: 7px;
  padding-right: 7px;
}

.gray {
  background-color: rgb(206, 206, 206);
  opacity: 0%;
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
  #main {
    display: flex;
    flex-direction: column;}
  .leftSide {
    align-items: center;
    margin-left: 25vw;
  }
    .left {
    margin-left: 25vw;
    animation-name: none;
    animation-duration: 1s;
    animation-fill-mode: forwards;
}
  .textbox {width: 80vw;}
  .controls {width: 81vw;}
  .rightSide {margin-top: 20px;}
 }

 </style>