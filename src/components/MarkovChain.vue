<template>
  <div id="markov-chain" :style="{ '--delay': controllerSettings ? controllerSettings.delay : 1000 }">
    <div class="padder"></div>
    <transition v-if="controllerSettings">
      <parser ref="activeStage" v-if="!states"
        :settings="controllerSettings"
        :data="data"
        @complete="states = $event"
      ></parser>
      <generator ref="activeStage" v-else
        :settings="controllerSettings"
        :data="data" :states="states"
      ></generator>
    </transition>
    <controller
      :mode="states ? 'generator' : 'parser'"
      @settings="controllerSettings = $event"
      @restart="forwardInstr('restart')"
      @nextStep="forwardInstr('nextStep')"
      @stopAnimation="forwardInstr('stopAnimation')"
      @clearData="clearData"
    ></controller>
  </div>
</template>

<script>
import Parser from './Parser'
import Generator from './Generator'
import Controller from './Controller'

export default {
  name: 'markov-chain',
  props: {
    data: Array
  },
  components: {
    Parser,
    Generator,
    Controller
  },
  data () {
    return {
      controllerSettings: null,
      states: null
    }
  },
  methods: {
    forwardInstr (instr) {
      this.$refs.activeStage.handle(instr)
    },
    clearData () {
      this.states = null
    }
  }
}
</script>

<style>
#markov-chain {
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  height: 100%;
}

#parser, #generator {
  height: 50%;
}

#current-name, #stopped-name, #possible-states, .word {
  font-family: 'Droid Sans Mono', monospace;
  font-size: 20px;
  transition: font-size 1s;
}

#current-name .finished .word {
  font-size: 28px;
}

#possible-states {
  font-size: 20px;
  margin-top: 20px;
  max-height: 50vh;
  position: relative;
}

#generator #possible-states {
  font-size: 16px;
  overflow: hidden;
}

.word,
.word .letter,
#possible-states .state,
.v-enter-active, .v-leave-active {
  transition: all calc(var(--delay) * 1ms);
}

.word {
  display: inline-block;
  margin-right: 20px;
}
.current-word .letter { display: inline-block; }
.current-word .letter.in-left-state {
  color: #edf177;
  font-weight: bold;
}
.current-word .letter.in-right-state {
  color: #90c9f5;
  font-weight: bold;
}
.current-word .letter.not-in-left-state + .letter.in-left-state { margin-left: 5px; }
.current-word .letter.in-left-state + .letter.not-in-left-state { margin-left: 5px; }
.current-word .letter.in-right-state + .letter.not-in-right-state { margin-left: 5px; }
.stopped-word { opacity: 0.5; }

#generator .placeholder {
  margin-left: -30px;
}

#possible-states .state {
  display: inline-block;
  margin-right: 20px;
}
#possible-states .state.selected {
  color: #fd8383;
  font-weight: bold;
  font-size: 20px;
}

.fade-absolute-container {
  position: relative;

  width: 100%;
}

.fade-absolute.v-enter-active, .fade-absolute.v-leave-active { width: 100%; }
.fade-absolute.v-enter-active, .fade-absolute.v-leave-active,
.fade-move.v-enter-active, .fade-move.v-leave-active { position: absolute; }

.v-enter, .v-leave-to { opacity: 0; }
.fade-absolute-top.v-enter { transform: translateY(2em); }
.fade-absolute-top.v-leave-to { transform: translateY(-2em); }
.fade-absolute-right.v-enter { transform: translateX(-4em); }
.fade-absolute-right.v-leave-to { transform: translateX(4em); }
.fade-move { transform: translate(calc(var(--moveX) * 1px), calc(var(--moveY) * 1px)); }
.fade-move.v-enter-to { transform: translate(0, 0); }
</style>
