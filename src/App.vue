<template>
  <div class="demo">
    <h1 class="demo__title">DateRangePicker 示例</h1>
    <p class="demo__desc">
      基于 Element UI 官方 <code>type="daterange"</code> 封装,保留了官方双面板并排显示、
      月份联动、一起开关的全部交互能力。<strong>增强点</strong>:点击面板顶部的「年月文字」
      即可弹出<strong>年代 + 年份 + 月份</strong>快速选择浮层,一步跳转到任意年月,
      解决原生只能用左右箭头逐月翻页的痛点。
    </p>

    <el-card shadow="never" class="demo__section">
      <div slot="header" class="demo__card-head">
        <span class="demo__card-title">① 对比:原生 daterange vs 自定义组件</span>
      </div>
      <el-row :gutter="24">
        <el-col :span="12">
          <div class="demo__cell">
            <div class="demo__cell-label">原生 type="daterange"</div>
            <el-date-picker
              v-model="nativeValue"
              type="daterange"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              value-format="yyyy-MM-dd"
              :picker-options="nativeShortcuts"
              unlink-panels
              style="width: 100%;"
            />
            <div class="demo__hint">头部年/月 <strong>不可</strong>点击,只能左右翻页</div>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="demo__cell">
            <div class="demo__cell-label">DateRangePicker(点击头部年月试试)</div>
            <date-range-picker
              v-model="customValue"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              value-format="yyyy-MM-dd"
              :picker-options="customShortcuts"
              unlink-panels
              clearable
              style="width: 100%;"
            />
            <div class="demo__value">值: {{ customValue }}</div>
            <div class="demo__hint">头部年/月 <strong>可</strong>点击快速跳转</div>
          </div>
        </el-col>
      </el-row>
    </el-card>

    <el-card shadow="never" class="demo__section">
      <div slot="header">② 特性演示(均为官方原生能力,开箱即用)</div>
      <el-row :gutter="24">
        <el-col :span="12">
          <div class="demo__cell">
            <div class="demo__cell-label">联动面板(unlink-panels=false)</div>
            <date-range-picker
              v-model="v1"
              range-separator="至"
              start-placeholder="开始"
              end-placeholder="结束"
              value-format="yyyy-MM-dd"
              style="width: 100%;"
            />
            <div class="demo__hint">左面板切月,右面板自动跟随下一月</div>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="demo__cell">
            <div class="demo__cell-label">禁选未来日期 + 自定义分隔符</div>
            <date-range-picker
              v-model="v2"
              range-separator="~"
              start-placeholder="开始"
              end-placeholder="结束"
              value-format="yyyy-MM-dd"
              :picker-options="{ disabledDate: disableFuture }"
              unlink-panels
              style="width: 100%;"
            />
          </div>
        </el-col>
      </el-row>
      <el-row :gutter="24" style="margin-top: 20px;">
        <el-col :span="12">
          <div class="demo__cell">
            <div class="demo__cell-label">禁用状态</div>
            <date-range-picker
              v-model="v3"
              range-separator="至"
              start-placeholder="开始"
              end-placeholder="结束"
              value-format="yyyy-MM-dd"
              disabled
              style="width: 100%;"
            />
          </div>
        </el-col>
        <el-col :span="12">
          <div class="demo__cell">
            <div class="demo__cell-label">带快捷选项 + 跳转增强</div>
            <date-range-picker
              v-model="v4"
              range-separator="至"
              start-placeholder="开始"
              end-placeholder="结束"
              value-format="yyyy-MM-dd"
              :picker-options="{ shortcuts: customShortcuts }"
              style="width: 100%;"
            />
            <div class="demo__hint">先点「最近一周」,再点头部年月快速跳转其它区间</div>
          </div>
        </el-col>
      </el-row>
    </el-card>

    <el-card shadow="never" class="demo__section">
      <div slot="header">③ 宽度自适应(支持自定义 width)</div>
      <div class="demo__width-row">
        <div class="demo__width-item">
          <span class="demo__width-tag">width: 200px</span>
          <date-range-picker
            v-model="w1"
            range-separator="至"
            start-placeholder="开始"
            end-placeholder="结束"
            value-format="yyyy-MM-dd"
            style="width: 200px;"
          />
        </div>
        <div class="demo__width-item">
          <span class="demo__width-tag">width: 280px</span>
          <date-range-picker
            v-model="w2"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            value-format="yyyy-MM-dd"
            style="width: 280px;"
          />
        </div>
        <div class="demo__width-item">
          <span class="demo__width-tag">默认(350px)</span>
          <date-range-picker
            v-model="w3"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            value-format="yyyy-MM-dd"
          />
        </div>
      </div>
      <div class="demo__tips">
        <i class="el-icon-info"></i>
        <span>设置 <code>style="width: 200px"</code> 后,开始/结束输入框会自动均分剩余空间,分隔符和图标不会被挤压。</span>
      </div>
    </el-card>

    <el-card shadow="never" class="demo__section">
      <div slot="header">用法说明</div>
      <pre class="demo__code">{{ usage }}</pre>
    </el-card>
  </div>
</template>

<script>
import DateRangePicker from './components/DateRangePicker.vue';

export default {
  name: 'App',
  components: { DateRangePicker },
  data() {
    return {
      nativeValue: [],
      customValue: [],
      v1: [],
      v2: [],
      v3: ['2024-01-01', '2024-01-31'],
      v4: [],
      w1: [],
      w2: [],
      w3: [],
      nativeShortcuts: [
        {
          text: '最近一周',
          onClick(picker) {
            const end = new Date();
            const start = new Date();
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
            picker.$emit('pick', [start, end]);
          }
        }
      ],
      customShortcuts: [
        {
          text: '最近一周',
          onClick(picker) {
            const end = new Date();
            const start = new Date();
            start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
            picker.$emit('pick', [start, end]);
          }
        },
        {
          text: '本月',
          onClick(picker) {
            const now = new Date();
            const start = new Date(now.getFullYear(), now.getMonth(), 1);
            picker.$emit('pick', [start, now]);
          }
        },
        {
          text: '今年',
          onClick(picker) {
            const now = new Date();
            const start = new Date(now.getFullYear(), 0, 1);
            picker.$emit('pick', [start, now]);
          }
        }
      ],
      usage: `<!-- 用法和原生 daterange 完全一致,所有 props 直通 -->
<template>
  <date-range-picker
    v-model="range"
    range-separator="至"
    start-placeholder="开始日期"
    end-placeholder="结束日期"
    value-format="yyyy-MM-dd"
    unlink-panels
    clearable
    :picker-options="{
      shortcuts: [...],
      disabledDate: date => date.getTime() > Date.now()
    }"
  />
</template>

<script>
import DateRangePicker from '@/components/DateRangePicker.vue';

export default {
  components: { DateRangePicker },
  data() {
    return { range: [] };
  }
};
<\/script>

<!-- 新增 prop:
  jumpable (Boolean, 默认 true):是否启用头部年月点击跳转 -->
`
    };
  },
  methods: {
    disableFuture(date) {
      return date.getTime() > Date.now();
    }
  }
};
</script>

<style lang="scss">
body {
  margin: 0;
  background: #f0f2f5;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.demo {
  max-width: 1080px;
  margin: 0 auto;
  padding: 32px 24px 48px;

  &__title {
    color: #303133;
    margin: 0 0 8px;
    font-size: 24px;
  }

  &__desc {
    color: #606266;
    line-height: 1.8;
    margin: 0 0 24px;
    font-size: 14px;

    code {
      background: #f5f7fa;
      padding: 2px 6px;
      border-radius: 3px;
      color: #e96900;
      font-size: 13px;
    }
  }

  &__section {
    margin-bottom: 20px;
  }

  &__card-head {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  &__card-title {
    font-weight: 600;
    color: #303133;
  }

  &__cell {
    background: #fafbfc;
    border: 1px solid #ebeef5;
    border-radius: 4px;
    padding: 16px;
    box-sizing: border-box;
  }

  &__cell-label {
    font-size: 13px;
    color: #909399;
    margin-bottom: 10px;
  }

  &__value {
    margin-top: 12px;
    color: #606266;
    font-size: 13px;
    word-break: break-all;
    min-height: 20px;
  }

  &__hint {
    margin-top: 8px;
    color: #c0c4cc;
    font-size: 12px;

    strong {
      color: #67c23a;
    }
  }

  &__width-row {
    display: flex;
    flex-wrap: wrap;
    gap: 24px;
    align-items: flex-start;
  }

  &__width-item {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 8px;
  }

  &__width-tag {
    font-size: 12px;
    color: #909399;
    background: #f4f4f5;
    padding: 2px 8px;
    border-radius: 3px;
  }

  &__tips {
    margin-top: 16px;
    padding: 10px 14px;
    background: #ecf5ff;
    border: 1px solid #d9ecff;
    border-radius: 4px;
    color: #409eff;
    font-size: 12.5px;
    line-height: 1.8;

    i {
      margin-right: 6px;
    }

    code {
      background: rgba(255, 255, 255, 0.6);
      padding: 1px 5px;
      border-radius: 3px;
      color: #e96900;
    }
  }

  &__code {
    background: #fafafa;
    padding: 16px;
    border-radius: 4px;
    font-family: 'Consolas', 'Monaco', monospace;
    font-size: 12.5px;
    line-height: 1.6;
    color: #2c3e50;
    overflow-x: auto;
    margin: 0;
  }
}
</style>
