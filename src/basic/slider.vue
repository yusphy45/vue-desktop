<template>
  <div>
    <div class="slider" :class="{'show-input': showInput}" @click="onSliderClick($event)" v-el:slider>
      <div class="slider-bar" :style="{width: currentPosition}"></div>
      <div class="slider-button" :style="{left: currentPosition}" v-el:button>
        <div class="pop-up" :class="{active: showTip}" v-el:pop-up>{{currentValue}}</div>
      </div>
    </div>
    <input type="text" v-if="showInput" :value="currentValue" @keyup="onInputChange()" v-el:input>
  </div>
</template>

<style>
  .slider {
    width: 100%;
    height: 5px;
    margin: 20px 0;
    background-color: #ccc;
    border-radius: 3px;
    position: relative;
    cursor: pointer;
    vertical-align: middle;
  }

  .slider.show-input {
    width: 88%;
    display: inline-block;
  }

  .slider + input {
    border: solid 1px #ccc;
    box-sizing: border-box;
    width: 8%;
    vertical-align: middle;
    margin-left: 3%;
    text-align: center;
  }

  .slider:hover .slider-button .pop-up {
    display: block;
  }

  .slider .slider-bar {
    height: 5px;
    background-color: #4997dc;
    border-top-left-radius: 3px;
    border-bottom-left-radius: 3px;
    position: absolute;
  }

  .slider .slider-button {
    height: 9px;
    width: 9px;
    border: solid 2px #4997dc;
    position: absolute;
    background-color: #fff;
    border-radius: 50%;
    top: -4px;
    transform: translateX(-50%);
    cursor: pointer;
  }

  .slider .slider-button .pop-up {
    font-size: 12px;
    line-height: 1.7;
    text-align: center;
    padding: 0 5px;
    border-radius: 4px;
    background-color: #666;
    color: #fff;
    position: absolute;
    bottom: 15px;
    left: -7px;
    cursor: default;
    display: none;
  }

  .slider .slider-button .pop-up.active {
     display: block;
  }
</style>

<script type="text/ecmascript-6" lang="babel">
  var getStyle = require('wind-dom').getStyle;
  
  export default {
    props: {
      min: {
        type: Number,
        default: 0
      },
      max: {
        type: Number,
        default: 100
      },
      step: {
        type: Number,
        default: 1
      },
      defaultValue: {
        type: Number,
        default: 0
      },
      showInput: {
        type: Boolean,
        default: false
      }
    },

    data() {
      return {
        showTip: false,
        dragging: false,
        newPos: null,
        oldValue: this.defaultValue,
        currentValue: this.defaultValue,
        currentPosition: (this.defaultValue - this.min) / (this.max - this.min) * 100 + '%'
      }
    },

    methods: {
      setPosition(newPos) {
        if (newPos >= 0 && (newPos <= 100)) {
          var lengthPerStep = 100 / ((this.max - this.min) / this.step);
          var steps = Math.round(newPos / lengthPerStep);
          this.currentValue = steps * lengthPerStep * (this.max - this.min) * 0.01 + this.min;
          this.currentPosition = (this.currentValue - this.min) / (this.max - this.min) * 100 + '%';
          if (!this.dragging) {
            if (this.currentValue !== this.oldValue) {
              this.$emit('change', this.currentValue);
              this.oldValue = this.currentValue;
            }
          }
        }
      },

      onSliderClick(event) {
        var currentX = event.clientX;
        var sliderOffsetLeft;
        getStyle(this.$el.parentNode, 'position') === 'static' ? sliderOffsetLeft = this.$els.slider.offsetLeft : sliderOffsetLeft = this.$el.parentNode.offsetLeft + this.$els.slider.offsetLeft;
        var newPos = (currentX - sliderOffsetLeft) / this.$sliderWidth * 100;
        this.setPosition(newPos);
      },

      onInputChange() {
        if (this.$els.input.value === '') {
          return;
        }
        if (!isNaN(this.$els.input.value)) {
          this.setPosition((this.$els.input.value - this.min) * 100 / (this.max - this.min));
        }
      }
    },

    computed: {
      $sliderWidth() {
        return (parseInt(getStyle(this.$els.slider, 'width')));
      }
    },

    compiled() {
      var startX = 0;
      var currentX = 0;
      var startPos = 0;
      var self = this;

      var onDragStart = function(event) {
        self.dragging = true;
        self.showTip = true;
        startX = event.clientX;
        startPos = parseInt(self.currentPosition);
      };

      var onDragging = function(event) {
        if (self.dragging) {
          currentX = event.clientX;
          var diff = (currentX - startX) / self.$sliderWidth * 100;
          self.newPos = startPos + diff;
          self.setPosition(self.newPos);
        }
      };

      var onDragEnd = function() {
        if (self.dragging) {
          self.dragging = false;
          self.showTip = false;
          self.setPosition(self.newPos);
          window.removeEventListener('mousemove', onDragging);
          window.removeEventListener('mouseup', onDragEnd);
        }
      };

      this.$els.button.addEventListener('mousedown', function(event) {
        onDragStart(event);
        window.addEventListener('mousemove', onDragging);
        window.addEventListener('mouseup', onDragEnd);
      });
    }
  }
</script>