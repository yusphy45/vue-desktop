<template>
  <div class="d-navmenu-item" :class="{ expanded: hasChild && expanded, active: !hasChild && active, toplevel: topLevel }">
    <div class="d-navmenu-item-label" @click="handleHeaderClick"><span v-if="icon"></span><a v-link="{ path: path }">{{text}}</a><span class="d-icon icon-arrow-right" v-if="hasChild"></span></div>
    <div class="d-navmenu-item-content" @click="$event.stopPropagation()" transition="navmenu" v-show="expanded">
      <slot></slot>
    </div>
  </div>
</template>

<style>
  .d-navmenu-item {
    box-sizing: border-box;
  }

  .d-navmenu-item.toplevel.expanded {
    border-left: 4px solid #19aa8d;
  }

  .d-navmenu-item.expanded {
    background-color: #f8f8f9;
  }

  .d-navmenu-item.expanded > .d-navmenu-item-content {
    display: block;
  }

  .d-navmenu-item.expanded > .d-navmenu-item-label .d-icon{
    transform: rotate(90deg);
  }

  .d-navmenu-item-label {
    position: relative;
    color: #676a6c;
    cursor: pointer;
  }

  .d-navmenu-item-label > a {
    display: inline-block;
    text-decoration: none;
    padding: 10px 15px 10px 20px;
    width: 100%;
    box-sizing: border-box;
    color: #676a6c;
  }

  .d-navmenu-item-label .d-icon {
    position: absolute;
    right: 15px;
    top: 9px;
    font-size: 12px;
    margin-top: 3px;
  }

  .d-navmenu-item-label:hover {
    font-weight: 600;
    color: #5b5d5f;
  }

  .d-navmenu-item-label:hover .d-icon {
    font-weight: normal;
  }

  .d-navmenu-item.expanded .d-navmenu-item-label > a {
    padding-left: 16px;
  }

  .d-navmenu-item-content {
    padding-left: 20px;
    display: none;
    overflow: hidden;
  }

  .navmenu-transition {
    transition: .3s;
  }
</style>

<script type="text/ecmascript-6" lang="babel">
  const domUtil = require('wind-dom');

  export default {
    props: {
      path: {},
      text: {},
      expanded: {
        type: Boolean
      },
      exclusive: {
        type: Boolean,
        default: false
      }
    },

    created() {
      this.$isNavMenu = true;
    },

    methods: {
      handleHeaderClick() {
        if (this.hasChild) {
          this.expanded = !this.expanded;
        }
      },

      handleExclusive(item) {
        if (this.exclusive) {
          var children = this.$children;
          children.forEach(function(child) {
            if (child !== item) {
              child.expanded = false;
            }
          });
        }
      }
    },

    watch: {
      expanded(newVal) {
        if (newVal) {
          this.$parent.handleExclusive(this);
        }
      }
    },

    data() {
      return {
        hasChild: false,
        topLevel: false
      }
    },

    ready() {
      this.hasChild = this.$children.length > 0;
      this.topLevel = !this.$parent.$isNavMenu;
    },

    transitions: {
      navmenu: {
        beforeEnter: function (el) {
          el.style.height = '0';
        },
        enter: function (el) {
          el.style.display = 'block';
          el.style.height = el.scrollHeight + 'px';
        },
        afterEnter: function(el) {
          el.style.display = '';
          el.style.height = '';
        },
        beforeLeave: function (el) {
          el.style.height = el.scrollHeight + 'px';
        },
        leave: function (el) {
          el.style.display = 'block';
          setTimeout(() => el.style.height = '0');
        },
        afterLeave: function (el) {
          el.style.display = '';
          el.style.height = '';
        }
      }
    }
  };
</script>