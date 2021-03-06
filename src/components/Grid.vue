<template>
  <div>
    <Controls></Controls>
    <div class="grid" ref="grid" :style="gridStyle">
      <div class="row" v-for="y in rows" :key="y">
          <Cell
            v-for="cell in cells[ y - 1 ]"
            :is="cell.type"
            :col="cell.x"
            :row="cell.y"
            :living="cell.alive"
            :key="cell.key"
            :style="cellStyle">
          </cell>
      </div>
    </div>
    <p style="text-align: center">Generation: {{ generation }}</p>
    <p style="text-align: center">( While stopped, clicking cells will toggle living state )</p>
  </div>
</template>

<script>
  import Cell         from './Cell';
  import Controls     from './Controls';
  import { eventBus } from '../helpers/EventBus';
  import * as helpers from '../helpers/cells';

  export default {
    data: function() {
      return {
        name: "Grid",
        width: helpers.grid_width,
        height: helpers.grid_height,
        cell_dims: helpers.cell_size,
        rows: helpers.rows,
        generation: 0,
        cells: helpers.reset(),
        next: null,
        rate: 400,
        intervalID: null,
        running: false,
        boardClear: true
      }
    },
    components: {
      Cell,
      Controls
    },
    computed: {
      gridStyle() {
        return {
          width: this.width+'px',
          height: this.height+'px'
        }
      },
      cellStyle() {
        return {
          width: this.cell_dims+'px',
          height: this.cell_dims+'px',
          border: '0.5px solid black'
        }
      }
    },
    methods: {
      start() {
        this.running = true;
        this.generation++;
        this.setNext();
        const vm = this;
        this.intervalID = setInterval( function() {
          vm.swap();
        }, vm.rate );
      },
      step() {
        if ( this.boardClear ) { return true; }
        else {
          this.setNext()
          this.swap();
        }
      },
      stop() {
        this.running = false;
        if ( this.intervalID ) {
          clearInterval( this.intervalID );
          this.intervalID = null;
        }
      },
      swap() {
        this.cells = this.next
        this.setNext();
        this.generation++;
      },
      setNext() {
        this.next = helpers.next( this.cells );
      }
    },
    created() {
      eventBus.$on( 'startEvent', () => {
        this.start();
      } );
      eventBus.$on( 'stepEvent', () => {
        this.step();
      } );
      eventBus.$on( 'stopEvent', () => {
        if ( this.running ) { this.stop(); }
      } );
      eventBus.$on( 'resetEvent', () => {
        if ( this.running ) { this.stop(); }
        if ( this.rate != 400 ) { this.rate = 400; }
        this.cells = helpers.reset();
        this.generation = 0;
        this.boardClear = true;
      } );
      eventBus.$on( 'randomizeEvent', () => {
        if ( this.running ) { this.stop(); }
        if ( this.rate != 400 ) { this.rate = 400; }
        this.cells = helpers.random();
        this.generation = 0;
        if ( this.boardClear ) { this.boardClear = false; }
      } );
      eventBus.$on( 'cellEvent', ( x, y ) => {
        if ( !this.running ) {
          this.cells = helpers.updateCells( x, y, this.cells );
          if ( this.generation == 0 ) {
            this.generation = 1;
          }
          if ( this.boardClear ) { this.boardClear = false; }
        }
      } );
      eventBus.$on( 'blinkerEvent', () => {
        if ( this.running ) { this.stop(); }
        if ( this.rate != 400 ) { this.rate = 400; }
        this.cells = helpers.createBlinker();
        this.generation = 0;
        if ( this.boardClear ) { this.boardClear = false; }
      } );
      eventBus.$on( 'beaconEvent', () => {
        if ( this.running ) { this.stop(); }
        if ( this.rate != 400 ) { this.rate = 400; }
        this.cells = helpers.createBeacon();
        this.generation = 0;
        if ( this.boardClear ) { this.boardClear = false; }
      } );
      eventBus.$on( 'pulsarEvent', () => {
        if ( this.running ) { this.stop(); }
        if ( this.rate != 400 ) { this.rate = 400; }
        this.cells = helpers.createPulsar();
        this.generation = 0;
        if ( this.boardClear ) { this.boardClear = false; }
      } );
      eventBus.$on( 'gosperEvent', () => {
        if ( this.running ) { this.stop(); }
        if ( this.rate != 400 ) { this.rate = 200; }
        this.cells = helpers.creatGosperGun();
        this.generation = 0;
        if ( this.boardClear ) { this.boardClear = false; }
      } );
    },
    beforeDestroy() {
      this.stop();
    }
  };
</script>

<style>
  .grid {
    background-color: rgba(134, 134, 134, 0.185);
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
    border: 0.1px solid rgba(82, 82, 82, 0.178);
    margin: 0 auto;
    margin-top: 5px;
    box-shadow: 0px 0px 120px 40px rgba(131, 1, 1, 0.445);
  }
  .row {
    display: flex;
    flex-direction: row;
    justify-content: space-evenly;
    width: 100%;
  }
</style>
