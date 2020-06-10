<template>
  <div>
    <Header :image="image"/>
    <div class="bingo-container">
      <div class="bingo-controls">
        <button class="bingo-button" @click="randomize" :disabled="locked">Arvo ruudukko!</button>
        <button
          class="bingo-button"
          @click="play"
          :disabled="selected.length !== 25"
          v-if="!locked"
        >Lukitse ja pelaa</button>
        <button
          class="bingo-button"
          @click="reset"
          :disabled="selected.length !== 25"
          v-if="locked"
        >Nollaa taulu</button>
      </div>
      <transition name="fade">
        <div class="bingo-confirmation-dialog" v-if="confirm">
          <div class="bingo-confirmation-window">
            <h3>Oletko varma?</h3>
            <p>Haluatko varmasti aloittaa bingon? Nykyään nuoriso täyttää näitä hirveällä kiireellä sen enempää ajattelematta oikeasti toimivia ruudukoita...</p>
            <div class="button-row">
              <button class="bingo-button" @click="cancel">Peruuta</button>
              <button class="bingo-button bingo-ok" @click="start">OK boomer</button>
            </div>
          </div>
        </div>
      </transition>
      <div class="bingo-grid" :class="{'bingo-grid--locked': locked}">
        <div class="bingo-row" v-for="(row, i) in bingoGrid.rows" :key="'row-' + i">
          <div class="bingo-square" v-for="(col, j) in row.columns" :key="'col-' + j">
            <div class="bingo-image-container" v-if="col.value">
              <img class="bingo-image" :src="col.value.img">
            </div>
            <v-select
              v-model="col.value"
              class="bingo-select"
              label="name"
              :options="options"
              :selectable="option => ! selected.includes(option.name)"
            ></v-select>
            <div class="bingo-label">{{ col.value ? col.value.name : '' }}</div>
            <input
              class="bingo-checkbox"
              type="checkbox"
              :id="'checked-' + i + j"
              v-model="crossed"
              :value="i + '/' + j"
            >
            <label class="bingo-cross" :for="'checked-' + i + j">
              <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 214 214" fill="currentColor">
                <polygon
                  fill-rule="evenodd"
                  points="82 132 82 232 132 232 132 132 232 132 232 82 132 82 132 -18 82 -18 82 82 -18 82 -18 132"
                  transform="rotate(45 107 107)"
                ></polygon>
              </svg>
            </label>
          </div>
        </div>
      </div>
      <transition name="fade">
        <div class="bingo-celebration" v-show="showBingo">
          <template v-if="isBingo < 12">
            <span v-if="isBingo > 1">{{isBingo}}x</span> bingo!
          </template>

          <template v-else>Ruudukko täynnä!</template>
        </div>
      </transition>
    </div>
  </div>
</template>

<script>
import animes from "../animes.json";
import Header from "./Header";

export default {
  name: "Bingo",
  components: {
    Header
  },
  methods: {
    randomize() {
      var animeList = JSON.parse(JSON.stringify(this.options));
      this.bingoGrid.rows.forEach(row => {
        row.columns.forEach(col => {
          animeList.sort(() => Math.random() - 0.5);
          col.value = animeList.pop();
        });
      });
    },
    play() {
      this.confirm = true;
    },
    start() {
      this.confirm = false;
      this.locked = true;
    },
    cancel() {
      this.confirm = false;
    },
    reset() {
      this.locked = false;
      this.crossed = [];
    },
    bingo() {
      var vm = this;
      vm.image =
        "https://media1.tenor.com/images/8dfa9e04bd20fb3ab560f5f7bd3c18b0/tenor.gif?itemid=15422354";
      setTimeout(function() {
        vm.showBingo = false;
        vm.image =
          "https://pbs.twimg.com/profile_images/1222632493712576518/eZ9Oad3o_400x400.jpg";
      }, 3000);
    }
  },
  computed: {
    selected() {
      var rows = this.bingoGrid.rows.map(a => a.columns);
      var squares = rows.flat();
      var values = squares.map(a => a.value);
      var filtered = values.filter(a => (a && a.name ? a.name : ""));
      var names = filtered.map(a => a.name);
      return names;
    },
    options() {
      var sortedAnimes = animes.sort(function(a, b) {
        var nameA = a.name.toUpperCase(); // ignore upper and lowercase
        var nameB = b.name.toUpperCase(); // ignore upper and lowercase
        if (nameA < nameB) {
          return -1;
        }
        if (nameA > nameB) {
          return 1;
        }

        // names must be equal
        return 0;
      });
      return sortedAnimes;
    },
    isBingo() {
      var checker = 0;
      var vm = this;
      if (vm.crossed.length >= vm.bingoGrid.rows.length) {
        // Check row bingos
        var rows = this.crossed.map(i => {
          return i.split("/")[0];
        });
        var rowCounts = {};
        rows.forEach(function(x) {
          rowCounts[x] = (rowCounts[x] || 0) + 1;
          if (rowCounts[x] === vm.bingoGrid.rows.length) {
            checker++;
          }
        });

        // Check column bingos
        var cols = vm.crossed.map(i => {
          return i.split("/")[1];
        });
        var colCounts = {};
        cols.forEach(function(x) {
          colCounts[x] = (colCounts[x] || 0) + 1;
          if (colCounts[x] === vm.bingoGrid.rows.length) {
            checker++;
          }
        });

        // Check diagonal
        var bingoDiagonals = [
          ["0/0", "1/1", "2/2", "3/3", "4/4"],
          ["4/0", "3/1", "2/2", "1/3", "0/4"]
        ];
        bingoDiagonals.forEach(function(x) {
          const intersection = x.filter(element =>
            vm.crossed.includes(element)
          );
          if (intersection.length === vm.bingoGrid.rows.length) {
            checker++;
          }
        });
      }
      if (checker > vm.bingos) {
        vm.showBingo = true;
        this.bingo();
      }
      vm.bingos = checker;
      return checker;
    }
  },
  data: function() {
    return {
      locked: false,
      image:
        "https://pbs.twimg.com/profile_images/1222632493712576518/eZ9Oad3o_400x400.jpg",
      bingos: 0,
      showBingo: false,
      confirm: false,
      crossed: [],
      bingoGrid: {
        rows: [
          {
            columns: [
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              }
            ]
          },
          {
            columns: [
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              }
            ]
          },
          {
            columns: [
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              }
            ]
          },
          {
            columns: [
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              }
            ]
          },
          {
            columns: [
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              },
              {
                value: ""
              }
            ]
          }
        ]
      }
    };
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
@import "vue-select/src/scss/vue-select.scss";

.fade-enter-active {
  transition: opacity 0.2s;
}

.fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}

.bingo-celebration {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  z-index: 20;
  font-size: 13vw;
  font-weight: 900;
  padding-top: 30px;
  color: white;
  -webkit-text-stroke: black 0.04em;
  pointer-events: none;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.75);
}

.bingo-confirmation-dialog {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  background: rgba(255, 255, 255, 0.6);
  z-index: 30;
}

.bingo-confirmation-window {
  padding: 20px 30px;
  margin: 60px auto 0;
  max-width: 80%;
  border: 2px solid black;
  background: white;
}

.button-row {
  display: flex;
  justify-content: space-between;
}

.bingo-container {
  position: relative;
  width: calc(95vw + 6px);
  max-width: 806px;
  margin: 0 auto;
}

.bingo-controls {
  display: flex;
  justify-content: space-between;
}

.bingo-button {
  background: white;
  border-radius: 4px;
  color: black;
  padding: 12px 20px;
  font-size: 18px;
  font-weight: bold;
  border: 2px solid #666;
  margin-bottom: 12px;
  &:hover {
    transform: scale(1.01);
    box-shadow: 0 0 0 1px #666;
  }

  &:focus {
    outline: 2px solid black;
    outline-offset: -4px;
  }
  &:active {
    transform: scale(0.99);
  }
  &:disabled {
    background: rgb(202, 202, 202);
    color: white;
    border: 2px solid rgb(202, 202, 202);
    cursor: not-allowed;

    &:hover,
    &:active {
      transform: none;
      box-shadow: none;
    }
  }
}

.bingo-grid {
  border: 3px solid black;
}

.bingo-row {
  height: 19vw;
  width: 95vw;
  max-width: 800px;
  max-height: 160px;
  display: flex;
}

.bingo-square {
  height: 19vw;
  max-height: 160px;
  width: 20%;
  border: 3px solid black;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  position: relative;
  word-break: break-word;
  &:nth-child(n + 4) {
    .vs__dropdown-menu {
      right: 0;
      left: auto;
    }
  }
}

.vs__dropdown-menu {
  width: auto;
  max-width: 60vw;
}

.bingo-image-container {
  position: absolute;
  width: 100%;
  height: 100%;
}

.bingo-image {
  object-fit: cover;
  width: 100%;
  height: 100%;
}

.bingo-select {
  width: 100%;
  background-color: rgba(255, 255, 255, 0.75);
  font-weight: bold;
  margin-top: 0px;
}

.bingo-label {
  display: none;
}

.bingo-checkbox {
  position: absolute;
  opacity: 0;
  display: none;
}

.bingo-cross {
  position: absolute;
  display: none;
  width: 100%;
  height: 100%;
}

.bingo-grid--locked {
  .bingo-select {
    display: none;
  }
  .bingo-label {
    width: 100%;
    background-color: rgba(255, 255, 255, 0.75);
    font-weight: bold;
    margin-top: 0px;
    z-index: 1;
    color: black;
    display: block;
    padding: 4px 2px;
  }
  .bingo-checkbox {
    display: block;
  }
  .bingo-cross {
    display: block;
    width: 100%;
    height: 100%;
    z-index: 2;
    position: absolute;
    color: #dd0000;
    &:hover {
      box-shadow: 0 0 3px 3px rgba(255, 0, 0, 0.5);
    }
    svg {
      display: none;
    }
  }
  .bingo-checkbox:checked + .bingo-cross svg {
    display: block;
    opacity: 0.85;
  }
}

.vs__dropdown-toggle {
  min-height: 33px;
}

.vs__search {
  height: 0;
  margin-top: -10px;
}

.vs--open .vs__search {
  height: auto;
}

@media screen and (max-width: 700px) {
  .vs__selected {
    font-size: 12px;
    margin: 0;
    padding: 4px 2px;
  }
  .vs__dropdown-toggle {
    flex-direction: column;
    max-height: 19vw;
    overflow: hidden;
  }
  .vs__clear {
    padding: 4px;
  }
  .vs__actions {
    width: 100%;
    justify-content: flex-end;
    padding: 0 8px;
  }
  .bingo-label {
    font-size: 14px;
  }
}

@media screen and (max-width: 450px) {
  .vs__selected {
    font-size: 11px;
    margin: 0;
    padding: 0;
    word-break: break-all;
  }
  .bingo-label {
    font-size: 12px;
  }
}
</style>
