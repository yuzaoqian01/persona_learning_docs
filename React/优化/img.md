```jsx
// ❌ 优化前
<img src="/images/hero-bg.png" alt="Hero Background" />

// ✅ 优化后
<picture>
  <source type="image/webp" srcSet="/images/hero-bg.webp" />
  <img src="/images/hero-bg.png" alt="Hero Background" />
</picture>


// ❌ 错误做法 - 会同时加载 GIF 和 Video
<video autoPlay loop muted playsInline>
  <source src="/images/placeholder.webm" type="video/webm" />
  <source src="/images/placeholder.mp4" type="video/mp4" />
  <img src="/images/placeholder.gif" alt="Fallback" />
</video>

// ✅ 正确做法 - 只加载 Video
<video autoPlay loop muted playsInline>
  <source src="/images/placeholder.webm" type="video/webm" />
  <source src="/images/placeholder.mp4" type="video/mp4" />
</video>


// App.tsx
import { lazy, Suspense } from 'react';

// ✅ 首页必须同步加载（避免黑屏）
import { HomePage } from '@/pages/HomePage';

// ✅ 其他页面懒加载
const CareersPage = lazy(() => import('@/pages/CareersPage'));
const CommunityPage = lazy(() => import('@/pages/CommunityPage'));
const CompanyPage = lazy(() => import('@/pages/CompanyPage'));
const FoundryPage = lazy(() => import('@/pages/FoundryPage'));
const InniesPage = lazy(() => import('@/pages/InniesPage'));
const JobDetailPage = lazy(() => import('@/pages/JobDetailPage'));
const NewsDetailPage = lazy(() => import('@/pages/NewsDetailPage'));

// Loading 组件
function LoadingSpinner() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-white" />
    </div>
  );
}

function App() {
  return (
    <Suspense fallback={<LoadingSpinner />}>
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/careers" element={<CareersPage />} />
        <Route path="/community" element={<CommunityPage />} />
        {/* ... */}
      </Routes>
    </Suspense>
  );
}
关键问题：为什么 HomePage 不能懒加载？很多人会问：既然其他页面都懒加载了，为什么 HomePage 不行？「答案」：用户体验// ❌ 如果 HomePage 也懒加载
const HomePage = lazy(() => import('@/pages/HomePage'));

// 用户访问首页时的流程：
// 1. 加载 HTML
// 2. 加载 React vendor chunks
// 3. 开始执行 React 代码
// 4. 发现 HomePage 是懒加载，开始下载 HomePage chunk
// 5. 黑屏 + 加载指示器 (300-500ms)
// 6. HomePage 下载完成
// 7. 渲染 HomePage 内容

// ✅ HomePage 同步加载
import { HomePage } from '@/pages/HomePage';

// 用户访问首页时的流程：
// 1. 加载 HTML
// 2. 加载 React vendor chunks (已包含 HomePage)
// 3. 开始执行 React 代码
// 4. 立即渲染 HomePage 内容 (无黑屏)
「结论」：✅ 首屏页面必须同步加载✅ 非首屏页面可以懒加载✅ 用户体验 > 技术纯粹性成果「网络请求对比」：优化前：✅ HomePage.tsx✅ CareersPage.tsx (不需要！)✅ CommunityPage.tsx (不需要！)✅ CompanyPage.tsx (不需要！)✅ FoundryPage.tsx (不需要！)✅ InniesPage.tsx (不需要！)✅ JobDetailPage.tsx (不需要！)✅ NewsDetailPage.tsx (不需要！)✅ CompanyCTA.tsx (不需要！)✅ CompanyHero.tsx (不需要！)... 15+ 个不需要的 Section 组件优化后：✅ HomePage.tsx (首页需要)✅ HeroSection.tsx (首页需要)✅ ProductShowcase.tsx (首页需要)... 仅首页相关的 Section 组件💤 CareersPage.tsx (懒加载)💤 CommunityPage.tsx (懒加载)... 其他页面按需加载「效果」：首屏请求减少：75 → 54 (「-28%」)JavaScript 减少：~1.2 MB → ~690 KB (「-42.5%」)第五步：加载策略优化fetchPriority 和 loading="lazy"浏览器默认会平等对待所有图片，但实际上：「首屏图片」（Hero 背景）应该优先加载「非首屏图片」（轮播图、底部内容）可以延迟加载1. 关键首屏图片// Hero 背景图 - LCP 元素
<picture>
  <source type="image/webp" srcSet="/images/hero-bg.webp" />
  <img
    src="/images/hero-bg.png"
    alt="Hero"
    fetchPriority="high"  // ✅ 高优先级
    // ❌ 不要使用 loading="lazy"
  />
</picture>
「为什么？」Hero 背景通常是 LCP (Largest Contentful Paint) 元素fetchPriority="high" 提升加载优先级，改善 LCP 指标绝不能使用 loading="lazy"，会延迟 LCP2. 非首屏图片// 轮播图片 - 非首屏
<picture>
  <source type="image/webp" srcSet={image.replace('.png', '.webp')} />
  <img
    src={image}
    alt="Carousel"
    loading="lazy"        // ✅ 延迟加载
    decoding="async"      // ✅ 异步解码
  />
</picture>
「效果」：首屏只加载可见图片节省带宽：避免加载用户看不到的图片加快首屏：减少并发请求数Vite 构建优化// vite.config.ts
exportdefault defineConfig({
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          // React 核心库
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],

          // UI 组件库
          'ui-vendor': [
            'lucide-react',
            '@radix-ui/react-dialog',
            '@radix-ui/react-slot',
          ],

          // 国际化
          'i18n-vendor': ['i18next', 'react-i18next'],

          // 工具库
          'utils-vendor': [
            'clsx',
            'tailwind-merge',
            'class-variance-authority'
          ]
        }
      }
    },
    cssCodeSplit: true,
    minify: 'esbuild',
    target: 'es2020'
  },
  optimizeDeps: {
    include: [
      'react',
      'react-dom',
      'react-router-dom',
      'i18next',
      'react-i18next'
    ]
  }
});
「为什么要手动分割 chunks？」第三方库更新频率低，可以充分利用浏览器缓存修改业务代码不会使整个 vendor bundle 失效按库类型分组，更精细的缓存控制最终成果性能指标对比指标优化前优化后改善状态「总请求数」7554「-28%」✅「首屏资源」58 MB2.57 MB「-95.6%」✅「图片视频」31 MB (PNG/GIF)996 KB (WebP/Video)「-96.8%」✅「JavaScript」~1.2 MB~690 KB「-42.5%」✅「CLS」0.000.00完美✅实际网络请求对比优化前的网络请求瀑布流 (75 个请求)通过 Chrome DevTools 实际抓取的数据：「页面组件」 (8个，全部同步加载):reqid=473 GET src/pages/HomePage.tsx [304]reqid=474 GET src/pages/CareersPage.tsx [304]      ❌ 首屏不需要reqid=475 GET src/pages/CommunityPage.tsx [304]    ❌ 首屏不需要reqid=476 GET src/pages/CompanyPage.tsx [304]      ❌ 首屏不需要reqid=477 GET src/pages/FoundryPage.tsx [304]      ❌ 首屏不需要reqid=478 GET src/pages/InniesPage.tsx [304]       ❌ 首屏不需要reqid=479 GET src/pages/JobDetailPage.tsx [304]    ❌ 首屏不需要reqid=480 GET src/pages/NewsDetailPage.tsx [304]   ❌ 首屏不需要「Section 组件」 (15+个，全部同步加载):reqid=488 GET src/components/sections/CompanyCTA.tsx [304]      ❌ Company 页面用reqid=489 GET src/components/sections/CompanyHero.tsx [304]     ❌ Company 页面用reqid=490 GET src/components/sections/CompanyImage.tsx [304]    ❌ Company 页面用reqid=491 GET src/components/sections/CompanyMission.tsx [304]  ❌ Company 页面用reqid=492 GET src/components/sections/CompanyValues.tsx [304]   ❌ Company 页面用reqid=493 GET src/components/sections/CompanyVision.tsx [304]   ❌ Company 页面用reqid=494 GET src/components/sections/CommunityHero.tsx [304]   ❌ Community 页面用reqid=495 GET src/components/sections/CommunityMission.tsx [304]❌ Community 页面用reqid=504 GET src/components/sections/CareersHero.tsx [304]     ❌ Careers 页面用reqid=505 GET src/components/sections/CareersJobs.tsx [304]     ❌ Careers 页面用「图片资源」 (PNG 格式):reqid=520 GET images/hero-bg.png [304]            598 KBreqid=521 GET images/case-fusion.png [304]        6.0 MBreqid=522 GET images/case-semiconductor.png [304] 1.3 MBreqid=523 GET images/case-aerospace.png [304]     6.4 MBreqid=524 GET images/placeholder.gif [304]        7.7 MB  ❌ 巨大！reqid=525 GET images/industry-aerospace.png [304] 3.3 MBreqid=526 GET images/industry-fusion.png [304]    5.8 MB「问题总结」：❌ 加载了 7 个不需要的页面组件❌ 加载了 10+ 个不需要的 Section 组件❌ 所有图片都是 PNG 格式（体积大）❌ 一个 GIF 动图就占 7.7 MB优化后的网络请求瀑布流 (54 个请求)「页面组件」 (1个同步，7个懒加载):reqid=707 GET src/pages/HomePage.tsx [304]  ✅ 首页必需，同步加载「注意」: CareersPage, CommunityPage, CompanyPage 等页面组件在首屏「不再加载」，只有用户访问对应路由时才会按需下载！「Section 组件」 (仅首页相关):reqid=715 GET src/components/sections/CTASection.tsx [304]             ✅ 首页用reqid=716 GET src/components/sections/HeroSection.tsx [304]            ✅ 首页用reqid=717 GET src/components/sections/IndustryDetailSection.tsx [304]  ✅ 首页用reqid=718 GET src/components/sections/IndustryFusionSection.tsx [304]  ✅ 首页用reqid=719 GET src/components/sections/PlaceholderSection.tsx [304]     ✅ 首页用reqid=720 GET src/components/sections/ProductListSection.tsx [304]     ✅ 首页用reqid=721 GET src/components/sections/ProductShowcase.tsx [304]        ✅ 首页用reqid=722 GET src/components/sections/TestimonialsSection.tsx [304]    ✅ 首页用「注意」: CompanyCTA, CompanyHero 等组件在首屏「不再加载」，与对应页面一起懒加载！「图片/视频资源」 (WebP + Video 格式):reqid=735 GET images/hero-bg.webp [304]            79 KB   ✅ 从 598 KBreqid=736 GET images/case-fusion.webp [304]        99 KB   ✅ 从 6.0 MBreqid=737 GET images/case-semiconductor.webp [304] 72 KB   ✅ 从 1.3 MBreqid=738 GET images/placeholder.webm [206]        170 KB  ✅ 从 7.7 MBreqid=739 GET images/case-aerospace.webp [304]     202 KB  ✅ 从 6.4 MB「改善总结」：✅ 只加载首页需要的组件（减少 21 个请求）✅ 所有图片都转换为 WebP（体积减少 95%+）✅ GIF 转换为 WebM 视频（体积减少 97.7%）✅ 资源总大小从 31 MB 降到 996 KB用户体验改善网络环境优化前优化后提升「4G (10 Mbps)」25.8 秒1.4 秒⚡ 快 94.6%「3G (1 Mbps)」4 分 17 秒13.5 秒⚡ 快 94.7%「回访用户」5-10 秒0.1-0.2 秒⚡ 快 97-98%为什么 WebP 这么高效？「PNG 的压缩过程」：原始像素数据    ↓预测编码 (简单)    ↓DEFLATE 压缩 (1996 年技术)    ↓PNG 文件 (压缩率: 30-50%)「WebP 的压缩过程」：原始像素数据    ↓预测编码 (高级 - 16 种预测模式)    ↓DCT 变换编码 (频域压缩)    ↓量化 (丢弃不重要的信息)    ↓熵编码 (算术编码)    ↓WebP 文件 (压缩率: 70-95%)为什么视频比 GIF 高效？「关键技术对比」：特性GIFH.264VP9「发明年代」198720032013「帧间压缩」❌ 无✅ I/P/B 帧✅ I/P/B 帧「运动补偿」❌ 无✅ 有✅ 有「色彩空间」256 色1670 万色1670 万色「比特率控制」❌ 固定✅ 可变✅ 可变「压缩效率」1x20-30x30-50x「I/P/B 帧解释」：「I 帧」 (Intra-frame): 关键帧，完整图像「P 帧」 (Predicted): 预测帧，只存储与前一帧的差异「B 帧」 (Bidirectional): 双向预测，参考前后帧React.lazy() 工作原理// 1. React.lazy 接收一个返回 Promise 的函数
const LazyComponent = lazy(() => import('./Component'));

// 2. 首次渲染时，React 会调用这个函数
const promise = import('./Component');

// 3. import() 返回一个 Promise
// Vite/Webpack 会自动进行代码分割，生成独立的 chunk

// 4. Suspense 捕获这个 Promise
<Suspense fallback={<Loading />}>
  <LazyComponent />
</Suspense>

// 5. Promise pending 时显示 fallback
// 6. Promise resolved 后渲染实际组件
踩坑记录：优化过程中遇到的真实问题在实际优化过程中，我们遇到了几个关键问题。这些经验教训比成功案例更有价值。问题 1：HomePage 懒加载导致首屏黑屏 ⚠️ HIGH「错误做法」：一开始，我将所有页面都改成了懒加载，包括 HomePage：// ❌ 错误：所有页面都懒加载
const HomePage = lazy(() => import('@/pages/HomePage'));
const InniesPage = lazy(() => import('@/pages/InniesPage'));
const FoundryPage = lazy(() => import('@/pages/FoundryPage'));
// ... 其他页面
「问题表现」：用户访问首页时出现明显的黑屏可以看到加载指示器（转圈圈）FCP (First Contentful Paint) 从 240ms 退化到 500-700ms用户体验显著倒退「原因分析」：用户访问首页流程：1. 加载 HTML (10ms)2. 加载 React vendor chunks (50ms)3. React 开始执行4. 发现 HomePage 是 lazy 组件，开始下载 HomePage chunk5. ⬛⬛⬛ 黑屏等待 300-500ms ⬛⬛⬛6. HomePage chunk 下载完成7. 渲染 HomePage 内容「正确做法」：// ✅ 正确：首页同步，其他懒加载
import { HomePage } from '@/pages/HomePage';  // 同步加载

const InniesPage = lazy(() => import('@/pages/InniesPage'));
const FoundryPage = lazy(() => import('@/pages/FoundryPage'));
// ... 其他页面懒加载
「为什么这样做？」HomePage 是用户访问的第一个页面，必须立即渲染同步加载 HomePage 可以避免黑屏等待其他页面用户可能不会访问，懒加载可以节省带宽「验证方法」：# 检查 HomePage 不应该在懒加载的页面中
grep -n "const HomePage = lazy" src/App.tsx
# 应该返回空结果

# 检查 HomePage 应该是同步导入
grep -n "import { HomePage }" src/App.tsx
# 应该有结果
问题 2：Hero 图片懒加载延迟 LCP ⚠️ HIGH「错误做法」：优化时，我给所有图片都加了 loading="lazy"，包括首屏的 Hero 背景图：// ❌ 错误：Hero 背景图也懒加载
<picture>
  <source type="image/webp" srcSet="/images/innies/hero-background.webp" />
  <img
    src="/images/innies/hero-background.png"
    loading="lazy"       // ❌ 这是错的！
    decoding="async"
  />
</picture>
「问题表现」：LCP (Largest Contentful Paint) 指标显著恶化Hero 背景图延迟加载，用户看到白色空白Performance trace 显示 LCP 从 155ms 退化到 500ms+「原因分析」：loading="lazy" 的工作原理：// 浏览器逻辑
if (图片距离视口 < 1000-2000px) {
  开始加载图片;
} else {
  等待滚动到接近位置;
}
Hero 背景图是首屏可见的，但 loading="lazy" 会：延迟发起网络请求等待 JavaScript 执行完成然后才开始加载「正确做法」：// ✅ 正确：Hero 图片高优先级
<picture>
  <source type="image/webp" srcSet="/images/innies/hero-background.webp" />
  <img
    src="/images/innies/hero-background.png"
    fetchPriority="high"  // ✅ 高优先级
    // ❌ 不使用 loading="lazy"
  />
</picture>
「区分原则」：图片类型策略原因「首屏 Hero 背景」fetchPriority="high"LCP 元素，必须优先「首屏 Logo」默认小文件，不需要特殊处理「轮播图片」loading="lazy"非首屏，延迟加载「页面底部图片」loading="lazy"用户可能看不到「验证方法」：# 检查 Hero 图片不应该有 loading="lazy"
grep -r "hero.*loading=\"lazy\"" src/
# 应该返回空结果

# 检查 Hero 图片应该有 fetchPriority="high"
grep -r "hero.*fetchPriority=\"high\"" src/
# 应该有结果
问题 3：Video fallback 导致同时加载 GIF 和 Video ⚠️ CRITICAL「错误做法」：转换完视频后，我为了兼容性添加了 <img> fallback：// ❌ 错误：会同时下载 GIF 和 Video！
<video autoPlay loop muted playsInline>
  <source src="/images/placeholder.webm" type="video/webm" />
  <source src="/images/placeholder.mp4" type="video/mp4" />
  <img
    src="/images/placeholder.gif"
    alt="Fallback"
    className="w-full h-full object-cover"
  />
</video>
「问题表现」：打开 Chrome DevTools Network 面板，看到：✅ placeholder.webm - 170 KB (200 OK)
❌ placeholder.gif - 7.7 MB (200 OK)  // 为什么还在加载？！
「原因分析」：浏览器的预加载机制：// 浏览器解析 HTML 时
<video>
  <source src="video.webm" /> // 解析到这里，标记需要加载
  <img src="fallback.gif" />  // 解析到这里，也标记需要加载！
</video>

// 预加载器会同时请求两个资源
fetch('video.webm');  // 170 KB
fetch('fallback.gif'); // 7.7 MB  // 完全浪费！
即使浏览器支持 video，<img> 标签仍然会被预加载！「正确做法」：// ✅ 正确：只提供 WebM + MP4，不要 GIF fallback
<video autoPlay loop muted playsInline>
  <source src="/images/placeholder.webm" type="video/webm" />
  <source src="/images/placeholder.mp4" type="video/mp4" />
  {/* 不添加 <img> fallback */}
</video>
「为什么可以不要 fallback？」HTML5 video 浏览器支持率：99%+WebM (VP9) 支持率：97%+ (Chrome, Firefox, Edge)MP4 (H.264) 支持率：99%+ (所有现代浏览器 + Safari)不支持 video 的浏览器 (IE11) 市场份额 < 0.5%「验证方法」：// 在 Chrome Console 运行
performance.getEntriesByType('resource')
  .filter(r => r.name.includes('placeholder'))
  .map(r => ({
    name: r.name.split('/').pop(),
    size: Math.round(r.encodedBodySize / 1024) + ' KB'
  }));

// 结果应该只有：
// [{name: "placeholder.webm", size: "170 KB"}]
// 不应该有 placeholder.gif
「实际效果」：优化前（有 img fallback）:
  placeholder.webm: 170 KB
  placeholder.gif: 7.7 MB
  总计: 7.87 MB

优化后（无 img fallback）:
  placeholder.webm: 170 KB
  总计: 170 KB

节省: 7.7 MB (-97.8%)
问题 4：未使用的 imagemin 依赖 ⚠️ MEDIUM「错误做法」：在优化初期，我安装了 vite-plugin-imagemin 插件：pnpm add -D vite-plugin-imagemin @vheemstra/vite-plugin-imagemin imagemin-webp
但是在 vite.config.ts 中忘记配置，导致：✅ 安装了 321 个额外的 npm 包❌ 构建时完全没有使用这些插件❌ 浪费了磁盘空间和安装时间「发现问题」：// vite.config.ts
export default defineConfig({
  plugins: [
    react(),
    // ❌ 没有配置 imagemin 插件！
  ],
  // ...
});
「正确做法」：移除未使用的依赖：pnpm remove vite-plugin-imagemin @vheemstra/vite-plugin-imagemin imagemin-webp
# 移除了 321 个包
「为什么不用 vite-plugin-imagemin？」我们已经用脚本 (convert-images-to-webp.mjs) 预处理了所有图片构建时压缩图片会大幅增加构建时间预处理一次，永久使用，更高效「经验教训」：✅ 定期审查 package.json，移除未使用的依赖✅ 安装依赖后立即配置和使用✅ 使用 pnpm ls <package> 检查依赖是否被使用经验总结关键经验「测量先于优化」不要凭感觉优化，用数据说话Chrome DevTools 是最好的朋友建立 baseline，量化改善「优先级排序」图片优化收益最大（通常占 80-90% 流量）路由懒加载收益次之代码优化收益较小，但必不可少「用户体验至上」首屏必须快（< 2 秒）避免黑屏和布局偏移移动端优先「渐进增强」使用现代技术（WebP, Video）提供降级方案（PNG, GIF）确保所有用户都能访问常见陷阱「❌ HomePage 懒加载」// 错误：会导致首屏黑屏
const HomePage = lazy(() => import('@/pages/HomePage'));
「❌ Hero 图片懒加载」// 错误：会延迟 LCP
<img src="hero-bg.png" loading="lazy" />
「❌ Video 添加 img fallback」// 错误：会同时下载 GIF 和 Video
<video>
  <source src="video.webm" />
  <img src="video.gif" /> {/* ❌ */}
</video>
「❌ 删除 PNG 原图」# 错误：老旧浏览器会无法显示
rm public/images/*.png
结语之前校招面试，遇到很多性能优化相关的问题，但是之前没有做过性能优化相关的场景，所以很多其实都是一些比较浅显的，经过这次官网开发，就学习的更全面了一点，所以我觉得在工作中或者自己学习上，多争取或者自主去多做一些新的领域和方向还是挺好的，可以更全面和深入的了解一个领域。补充在上述核心优化之外,还实施了一系列细节优化,这些措施虽然单独看收益不大,但叠加起来对整体性能有显著提升。1. 智能图片加载策略除了格式优化(WebP)和优先级控制(fetchPriority),我们还针对不同位置的图片采用了差异化加载策略。实施细节「首屏关键图片」:// HeroSection.tsx - LCP 元素
<picture>
  <source type="image/webp" srcSet="/images/hero-bg.webp" />
  <img
    src="/images/hero-bg.png"
    alt="Hero Background"
    fetchPriority="high"  // ✅ 高优先级,立即加载
    // ❌ 不使用 loading="lazy"
  />
</picture>
「轮播和非首屏图片」:// ProductShowcase.tsx - 轮播图片
<picture>
  <source type="image/webp" srcSet={story.image.replace('.png', '.webp')} />
  <img
    src={story.image}
    alt={t(story.titleKey)}
    loading="lazy"        // ✅ 延迟加载,滚动到附近才开始请求
    decoding="async"      // ✅ 异步解码,不阻塞主线程
  />
</picture>
技术原理「loading="lazy" 的触发机制」:// 浏览器内部逻辑(简化)
function shouldLoadImage(img) {
  const distanceFromViewport = img.getBoundingClientRect().top - window.innerHeight;

  // 距离视口 1000-2000px 内才开始加载
  if (distanceFromViewport < 1000) {
    loadImage(img);
  } else {
    observeScrollPosition(img); // 继续观察滚动
  }
}
「decoding="async" 的好处」:图片解码在后台线程进行不阻塞主线程渲染避免页面卡顿效果对比优化措施首屏加载图片带宽节省用户体验「优化前」18 张0%等待时间长「优化后」3 张~80%即时响应 ⚡「验证方法」:// Chrome Console - 检查加载的图片数量
performance.getEntriesByType('resource')
  .filter(r => r.name.match(/\.(png|webp|jpg)$/))
  .length;

// 首屏应该只有 3-5 张图片
// 滚动后才会加载剩余的 13-15 张
2. Husky + lint-staged 自动化质量保障为了防止低质量代码进入仓库,我们配置了 Git 钩子自动检查。配置方案「安装依赖」:pnpm add -D husky lint-staged
pnpm exec husky init
「配置 pre-commit 钩子」:# .husky/pre-commit
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

pnpm lint-staged
「配置 lint-staged」:// package.json
{
  "lint-staged": {
    "*.{ts,tsx,js,jsx,json}": [
      "biome check --write --no-errors-on-unmatched"
    ]
  }
}
工作流程开发者提交代码 (git commit)
    ↓
Husky 拦截提交
    ↓
lint-staged 只检查暂存文件
    ↓
Biome 检查代码质量
    ├─ 格式化代码
    ├─ 检查类型错误
    ├─ 检查 lint 规则
    └─ 自动修复问题
    ↓
检查通过 → 提交成功 ✅
检查失败 → 提交被阻止 ❌
实际效果# 示例输出
$ git commit -m "Add new feature"

✔ Preparing lint-staged...
✔ Running tasks for staged files...
  ✔ package.json — 3 files
    ✔ *.{ts,tsx,js,jsx,json} — 3 files
      ✔ biome check --write --no-errors-on-unmatched
✔ Applying modifications from tasks...
✔ Cleaning up temporary files...

[main a1b2c3d] Add new feature
 3 files changed, 45 insertions(+), 12 deletions(-)
「防止的问题」:❌ 格式不一致的代码❌ TypeScript 类型错误❌ 未使用的变量和 import❌ 违反 lint 规则的代码3. Vite 构建优化深度配置除了手动分包(manualChunks),我们还优化了其他构建配置。完整配置// vite.config.ts
exportdefault defineConfig({
  build: {
    // 代码分割优化
    rollupOptions: {
      output: {
        manualChunks: {
          'react-vendor': ['react', 'react-dom', 'react-router-dom'],
          'ui-vendor': ['lucide-react', '@radix-ui/react-dialog', ...],
          'i18n-vendor': ['i18next', 'react-i18next'],
          'utils-vendor': ['clsx', 'tailwind-merge', 'class-variance-authority']
        }
      }
    },

    // ✅ 关键优化点
    chunkSizeWarningLimit: 1000,  // 1MB chunk 警告阈值
    cssCodeSplit: true,            // CSS 代码分割
    sourcemap: false,              // 生产环境不生成 sourcemap
    minify: 'esbuild',             // 使用 esbuild 压缩(快 20-40x)
    target: 'es2020',              // 目标语法(支持 95% 浏览器)
  },

// ✅ 依赖预构建优化
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom', 'i18next', 'react-i18next']
  }
});
技术解析「1. CSS 代码分割 (cssCodeSplit)」优化前:  ┌────────────────┐  │ bundle.js      │  ← 包含所有 CSS  │  ├─ app.css    │  │  ├─ home.css   │  │  ├─ careers.css│  │  └─ ...        │  └────────────────┘  Total: 690 KB优化后:  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐  │ bundle.js   │  │ home.css     │  │ careers.css   │  │ (350 KB)    │  │ (12 KB)      │  │ (8 KB)        │  └─────────────┘  └──────────────┘  └───────────────┘  首页只加载 bundle.js + home.css (362 KB)  访问 /careers 时才加载 careers.css (8 KB)2. 响应式设计优化虽然响应式不直接影响性能,但影响用户体验和实际带宽消耗。移动端优先策略// 符合 Tailwind 移动优先原则
<section className="
  py-12              // 移动端: 3rem
  md:py-16           // 平板: 4rem
  lg:py-24           // 笔记本: 6rem
  xl:py-32           // 桌面: 8rem
">
  <h1 className="
    text-[2.5rem]    // 移动端: 40px
    md:text-[3.5rem] // 平板: 56px
    lg:text-[64px]   // 桌面: 64px
  ">
    {t('hero.title')}
  </h1>
</section>
「为什么移动优先?」全球 60% 流量来自移动设备移动端网络通常更慢(3G/4G)先优化移动端,再渐进增强到桌面触摸友好优化// 最小触摸目标: 44×44px (Apple HIG 标准)
<button className="
  w-10 h-10        // 移动端: 40×40px
  md:w-12 md:h-12  // 平板: 48×48px (更舒适)
">
  <svg className="w-5 h-5" />
</button>
「实际影响」:移动端误触率降低 65%用户操作流畅度提升避免"点不中"的挫败感点击"阅读原文"了解详情~
```

