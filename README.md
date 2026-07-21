# DateRangePicker

基于 Element UI 官方 `type="daterange"` 封装,保留所有原生能力,新增**头部年月快速跳转**功能。

## 问题

原生 `el-date-picker type="daterange"` 只能用左右箭头逐月翻页,点击面板顶部的「年份」「月份」无反应。

## 解决方案

保持官方 daterange 的所有优秀交互(双面板并排、月份联动、一起开关、`unlink-panels`、`shortcuts` 等),只做一处增强:

点击面板顶部的「年月文字」,弹出「年代切换 + 年份选择 + 12 月份」快速选择浮层,一步跳转到任意年月。

## 特性

✅ 双面板并排显示、一起开关(官方原生)  
✅ 月份联动(`unlink-panels` 关闭时,切左面板月份,右面板自动下一月)  
✅ 头部年月可点击,弹出快速选择浮层(增强)  
✅ 浮层:年代切换 `« 2020-2029 »` + 年份网格(高亮当前) + 12 月按钮  
✅ 点击年份切换焦点年,点击月份立即跳转  
✅ 宽度自适应:设置 `style="width:200px"`,输入框自动均分,不会溢出  
✅ 全量透传官方所有 props/事件/插槽  
✅ 完整支持 `shortcuts`、`disabled-date`、`range-separator` 等

## 安装

```bash
npm install el-date-range --save
```

## 用法

```vue
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
import DateRangePicker from 'el-date-range';

export default {
  components: { DateRangePicker },
  data() {
    return {
      range: []
    };
  }
};
</script>
```

## Props

与官方 `el-date-picker type="daterange"` 完全一致,自动透传。新增一个可选 prop:

| 属性 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| jumpable | Boolean | true | 是否启用头部年月点击跳转 |

## 事件

所有官方事件自动透传:`change`、`pick`、`blur`、`focus` 等。

## 快速选择浮层

点击面板任意一侧的「年月文字」弹出:

```
‹  2020 - 2029  ›      ← 年代切换
2020 2021 2022 2023      ← 年份选择(高亮当前年)
2024 2025 2026 2027
2028 2029
───────────────────────────
 1月  2月  3月  4月       ← 月份选择
 5月  6月  7月  8月
 9月 10月 11月 12月
```

- 点击年份:切换焦点年,高亮同步
- 点击月份:跳转到 `焦点年 + 选中月`,关闭浮层
- 年份与月份联动:当年份不匹配当前面板年份时,月份高亮自动清除

## 对比官方 `type="date"`

| | 官方 `type="date"` | 本组件 |
|---|---|---|
| 面板 | 单个 | 双个并排(daterange) |
| 年月切换 | 点击进入 year/month 视图,再选,需要 2 步 | 点击弹出浮层,年+月 1 步直达 |
| 月份联动 | 无 | 有(可选 `unlink-panels` 关闭) |
| 快捷选项 | 有 | 有(官方原生) |

## 开发

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run serve

# 构建
npm run build
```

## 依赖

- Vue 2.x
- Element UI 2.x

## License

MIT

---

本项目基于 [Element UI](https://github.com/ElemeFE/element) 修改,遵循 MIT 协议。