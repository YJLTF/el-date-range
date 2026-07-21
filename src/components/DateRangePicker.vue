<template>
  <div class="drp-root">
    <el-date-picker
      ref="picker"
      type="daterange"
      v-bind="$attrs"
      :value="value"
      v-on="forwardListeners"
      @input="onInput"
    />
    <div
      v-if="jumper.visible"
      ref="jumper"
      class="drp-jumper"
      :style="jumperStyle"
      @mousedown="onJumperMousedown"
      @mouseup="onJumperMousedown"
    >
      <div class="drp-jumper__nav">
        <button type="button" class="drp-jumper__nav-btn" @click="shiftDecade(-1)">«</button>
        <span class="drp-jumper__nav-label">{{ decadeStart }} - {{ decadeStart + 9 }}</span>
        <button type="button" class="drp-jumper__nav-btn" @click="shiftDecade(1)">»</button>
      </div>
      <div class="drp-jumper__grid">
        <button
          v-for="y in years"
          :key="y"
          type="button"
          class="drp-jumper__cell"
          :class="{ 'is-active': y === activeYear }"
          @click="pickYear(y)"
        >{{ y }}
        </button>
      </div>
      <div class="drp-jumper__divider"></div>
      <div class="drp-jumper__grid">
        <button
          v-for="m in 12"
          :key="m"
          type="button"
          class="drp-jumper__cell"
          :class="{ 'is-active': (m - 1) === activeMonth }"
          @click="pickMonth(m - 1)"
        >{{ m }}月
        </button>
      </div>
      <div class="drp-jumper__hint">点击年份切换 • 点击月份跳转</div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'DateRangePicker',
  inheritAttrs: false,
  model: {
    prop: 'value',
    event: 'input'
  },
  props: {
    value: {
      type: Array,
      default: () => []
    },
    jumpable: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      jumper: {
        visible: false,
        side: 'left',
        rect: null,
        year: new Date().getFullYear()
      },
      _unwatch: null,
      _docMousedown: null
    };
  },
  computed: {
    forwardListeners() {
      const rest = Object.assign({}, this.$listeners);
      delete rest.input;
      return rest;
    },
    decadeStart() {
      return Math.floor(this.jumper.year / 10) * 10;
    },
    years() {
      const start = this.decadeStart;
      const arr = [];
      for (let i = 0; i < 10; i++) arr.push(start + i);
      return arr;
    },
    activeYear() {
      return this.jumper.year;
    },
    activeMonth() {
      const panel = this.panelInstance;
      if (!panel) return null;
      const d = this.jumper.side === 'left' ? panel.leftDate : panel.rightDate;
      if (!d) return null;
      if (d.getFullYear() !== this.jumper.year) return null;
      return d.getMonth();
    },
    panelInstance() {
      const p = this.$refs.picker;
      return p && p.picker;
    },
    jumperStyle() {
      if (!this.jumper.rect) return {display: 'none'};
      const r = this.jumper.rect;
      const w = 248;
      let left = r.left;
      if (left + w > window.innerWidth - 8) {
        left = window.innerWidth - w - 8;
      }
      if (left < 8) left = 8;
      const top = r.bottom + 6;
      return {
        top: top + 'px',
        left: left + 'px'
      };
    }
  },
  watch: {
    jumpable(v) {
      if (!v) this.closeJumper();
    }
  },
  mounted() {
    this.$nextTick(this.setupEnhance);
  },
  beforeDestroy() {
    this.teardownEnhance();
    this.closeJumper();
  },
  methods: {
    onInput(val) {
      this.$emit('input', val);
    },
    setupEnhance() {
      const p = this.$refs.picker;
      if (!p) return;
      this._unwatch = this.$watch(
        () => p.pickerVisible,
        (visible) => {
          if (visible && this.jumpable) {
            this.$nextTick(() => this.hookPanel());
          } else {
            this.closeJumper();
          }
        }
      );
    },
    teardownEnhance() {
      if (this._unwatch) {
        this._unwatch();
        this._unwatch = null;
      }
    },
    hookPanel() {
      const panel = this.panelInstance;
      if (!panel || !panel.$el) return;
      const el = panel.$el;
      if (el.__drpHooked) return;
      el.__drpHooked = true;
      const sides = [
        {side: 'left', selector: '.is-left .el-date-range-picker__header'},
        {side: 'right', selector: '.is-right .el-date-range-picker__header'}
      ];
      sides.forEach(({side, selector}) => {
        const header = el.querySelector(selector);
        if (!header) return;
        const label = header.querySelector('div');
        if (!label) return;
        label.classList.add('drp-jumpable');
        label.addEventListener('click', (ev) => this.onLabelClick(side, ev));
      });
    },
    onLabelClick(side, ev) {
      ev.stopPropagation();
      ev.preventDefault();
      const panel = this.panelInstance;
      if (!panel) return;
      const d = side === 'left' ? panel.leftDate : panel.rightDate;
      this.jumper.side = side;
      this.jumper.year = d ? d.getFullYear() : new Date().getFullYear();
      this.jumper.rect = ev.target.getBoundingClientRect();
      this.jumper.visible = true;
      this.$nextTick(this.bindDocMousedown);
    },
    bindDocMousedown() {
      if (this._docMousedown) return;
      this._docMousedown = (e) => {
        const pop = this.$refs.jumper;
        if (pop && pop.contains(e.target)) return;
        this.closeJumper();
      };
      document.addEventListener('mousedown', this._docMousedown, true);
    },
    closeJumper() {
      this.jumper.visible = false;
      if (this._docMousedown) {
        document.removeEventListener('mousedown', this._docMousedown, true);
        this._docMousedown = null;
      }
    },
    shiftDecade(delta) {
      this.jumper.year = this.decadeStart + delta * 10;
    },
    pickYear(y) {
      this.jumper.year = y;
    },
    pickMonth(m) {
      const panel = this.panelInstance;
      if (!panel) return;
      const side = this.jumper.side;
      const y = this.jumper.year;
      const newDate = new Date(y, m, 1);
      if (side === 'left') {
        panel.leftDate = newDate;
        if (!panel.unlinkPanels) {
          panel.rightDate = new Date(y, m + 1, 1);
        }
      } else {
        panel.rightDate = newDate;
        if (!panel.unlinkPanels) {
          panel.leftDate = new Date(y, m - 1, 1);
        }
      }
      this.closeJumper();
    },
    onJumperMousedown(e) {
      e.stopPropagation();
    }
  }
};
</script>

<style lang="scss">
.drp-root {
  display: inline-block;

  .el-date-editor.el-range-editor.el-input__inner {
    width: 100%;
    box-sizing: border-box;
    flex-wrap: nowrap;
  }

  .el-range__icon {
    flex: 0 0 auto;
    float: none;
  }

  .el-range-input {
    width: auto !important;
    flex: 1 1 0;
    min-width: 0;
  }

  .el-range-separator {
    flex: 0 0 auto;
    width: auto !important;
    min-width: 22px;
    padding: 0 4px;
  }

  .el-range__close-icon {
    flex: 0 0 auto;
    float: none;
  }
}

.el-date-range-picker__header div.drp-jumpable {
  cursor: pointer;
  transition: color 0.15s;

  &:hover {
    color: #409eff;
  }
}

.drp-jumper {
  position: fixed;
  z-index: 3000;
  width: 248px;
  background: #fff;
  border: 1px solid #e4e7ed;
  border-radius: 4px;
  box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.12);
  padding: 12px;
  box-sizing: border-box;
  user-select: none;
  font-size: 12px;
  color: #606266;

  &__nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 8px;
  }

  &__nav-btn {
    width: 20px;
    height: 20px;
    line-height: 18px;
    text-align: center;
    border: none;
    background: transparent;
    color: #909399;
    cursor: pointer;
    border-radius: 3px;
    padding: 0;

    &:hover {
      color: #409eff;
    }
  }

  &__nav-label {
    font-size: 13px;
    color: #303133;
    font-weight: 500;
  }

  &__grid {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 4px;
  }

  &__cell {
    height: 30px;
    line-height: 28px;
    text-align: center;
    border: 1px solid transparent;
    border-radius: 3px;
    cursor: pointer;
    color: #606266;
    background: transparent;

    &:hover {
      color: #409eff;
      background: #f5f7fa;
    }

    &.is-active {
      color: #fff;
      background: #409eff;

      &:hover {
        color: #fff;
        background: #409eff;
      }
    }
  }

  &__divider {
    height: 1px;
    background: #ebeef5;
    margin: 8px 0;
  }

  &__hint {
    margin-top: 8px;
    text-align: center;
    color: #c0c4cc;
    font-size: 11px;
  }
}
</style>
