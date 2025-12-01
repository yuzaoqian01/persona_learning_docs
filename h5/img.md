# 使用

```css
<img 
  src="image-800w.jpg"
  srcset="image-320w.jpg 320w,
          image-480w.jpg 480w,
          image-800w.jpg 800w"
  sizes="(max-width: 600px) 100vw,
         (max-width: 1200px) 50vw,
         33vw"
  alt="响应式图片示例"
>

```

```css
<picture>
  <!-- 桌面端：宽屏全景 -->
  <source media="(min-width: 1200px)" srcset="hero-desktop.jpg">
  <!-- 平板端：适中裁剪 -->
  <source media="(min-width: 768px)" srcset="hero-tablet.jpg">
  <!-- 移动端：竖版特写 -->
  <img src="hero-mobile.jpg" alt="产品展示">
</picture>

<picture>
  <source type="image/avif" srcset="image.avif">
  <source type="image/webp" srcset="image.webp">
  <img src="image.jpg" alt="格式优化示例">
</picture>

<picture>
  <!-- 大屏 + AVIF -->
  <source media="(min-width: 1200px)" type="image/avif" srcset="large.avif">
  <!-- 大屏 + WebP -->
  <source media="(min-width: 1200px)" type="image/webp" srcset="large.webp">
  <!-- 大屏降级 -->
  <source media="(min-width: 1200px)" srcset="large.jpg">
  
  <!-- 移动端方案 -->
  <img src="small.jpg" alt="复杂条件图片">
</picture>



```

