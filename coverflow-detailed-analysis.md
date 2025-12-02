# Coverflow Carousel 详细分析

## 从 page-88920c3a40ea888b.js 提取的关键代码

```javascript
function o(e) {
  let { src: n, index: t } = e;
  let { offset: o, props: l } = useCarousel(); // i.$H 是 useCarousel hook

  // rotateY: [-200, 0, 200] → [20, 0, -20]
  // 左侧图片向右旋转（正角度），右侧图片向左旋转（负角度）
  let c = useTransform(o, [-200, 0, 200], [20, 0, -20]);

  // scale: [-200, 0, 200] → [0.7, 1, 0.7]
  // 中间图片 scale=1，左右图片 scale=0.7（压缩效果）
  let m = useTransform(o, [-200, 0, 200], [0.7, 1, 0.7]);

  // x: [-800, -200, 200, 800] → ["100%", "0%", "0%", "-100%"]
  // 左侧图片向右偏移（正百分比），右侧图片向左偏移（负百分比）
  let h = useTransform(
    o,
    [-800, -200, 200, 800],
    ["100%", "0%", "0%", "-100%"]
  );

  // zIndex: 基于 offset 绝对值计算
  let g = useTransform(o, (e) => Math.max(0, Math.round(1000 - Math.abs(e))));

  return (
    <li style={{ zIndex: g }}>
      <img
        style={{
          transformPerspective: 500,
          x: h, // 水平偏移，造成压缩的视觉效果
          rotateY: c, // 3D 旋转，左右方向不同
          scale: m, // 缩放，中间1，两侧0.7
        }}
      />
    </li>
  );
}
```

## 关键发现

1. **offset 的含义**：

   - offset 是每个 item 相对于容器中心的位置（像素值）
   - 中心 item: offset = 0
   - 左侧 item: offset < 0（负值）
   - 右侧 item: offset > 0（正值）

2. **压缩效果的实现**：

   - **scale**: 中间 1，两侧 0.7（垂直和水平都压缩）
   - **x 偏移**: 左侧向右偏移（"100%" 到 "0%"），右侧向左偏移（"0%" 到 "-100%"）
   - **rotateY**: 左侧向右旋转（20 度），右侧向左旋转（-20 度）
   - 这三个效果组合起来，造成左右图片的压缩和方向不同的视觉效果

3. **显示 5 张图**：

   - 容器宽度 350px，item 宽度 350px
   - 通过 mask 渐变遮罩，只显示中间区域
   - 大约可以看到 5 张图（中间 1 张 + 左右各 2 张）

4. **useTransform 的映射逻辑**：

   - `useTransform(input, inputRange, outputRange)` 是线性插值
   - 例如 `[-200, 0, 200] → [20, 0, -20]` 表示：
     - offset = -200 时，rotateY = 20
     - offset = 0 时，rotateY = 0
     - offset = 200 时，rotateY = -20
     - 中间值线性插值

5. **x 偏移的特殊性**：
   - `["100%", "0%", "0%", "-100%"]` 中的百分比是相对于 item 自身的宽度
   - 左侧图片：offset 从 -800 到 -200，x 从 "100%" 到 "0%"
   - 中间图片：offset 从 -200 到 200，x = "0%"
   - 右侧图片：offset 从 200 到 800，x 从 "0%" 到 "-100%"
