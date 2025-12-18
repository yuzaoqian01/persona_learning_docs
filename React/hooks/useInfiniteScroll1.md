```ts
import { useEffect, useRef } from 'react'

// ---------- canUseIO ----------
export const canUseIO = (() => {
  if (typeof window === 'undefined') return false
  return (
    'IntersectionObserver' in window &&
    'IntersectionObserverEntry' in window &&
    'intersectionRatio' in IntersectionObserverEntry.prototype &&
    'isIntersecting' in IntersectionObserverEntry.prototype
  )
})()

// ---------- throttle ----------
export function throttle<T extends (...args: any[]) => void>(fn: T, wait = 200) {
  let lastTime = 0
  let timer: number | null = null

  return function (this: any, ...args: Parameters<T>) {
    const now = Date.now()
    const remaining = wait - (now - lastTime)

    if (remaining <= 0) {
      if (timer) {
        clearTimeout(timer)
        timer = null
      }
      lastTime = now
      fn.apply(this, args)
    } else if (!timer) {
      timer = window.setTimeout(() => {
        lastTime = Date.now()
        timer = null
        fn.apply(this, args)
      }, remaining)
    }
  }
}

// ---------- useInfiniteScroll ----------
interface Options {
  disabled?: boolean
  noMore?: boolean
  root?: Element | null
  rootMargin?: string
  threshold?: number | number[]
}

export function useInfiniteScroll(
  sentinelRef: React.RefObject<Element>,
  loadMore: () => void,
  options: Options = {}
) {
  const {
    disabled = false,
    noMore = false,
    root = null,
    rootMargin = '150px',
    threshold = 0,
  } = options

  const loadMoreRef = useRef(loadMore)
  loadMoreRef.current = loadMore

  const lockedRef = useRef(false)

  useEffect(() => {
    if (disabled || noMore) return
    const el = sentinelRef.current
    if (!el) return

    const tryLoad = () => {
      if (lockedRef.current || noMore) return
      lockedRef.current = true
      try {
        loadMoreRef.current()
      } finally {
        // 解锁由外部请求完成后再次触发
        setTimeout(() => {
          lockedRef.current = false
        }, 0)
      }
    }

    // ---------- IntersectionObserver ----------
    if (canUseIO) {
      const io = new IntersectionObserver(
        ([entry]) => {
          if (entry.isIntersecting) {
            tryLoad()
          }
        },
        { root, rootMargin, threshold }
      )

      io.observe(el)
      return () => io.disconnect()
    }

    // ---------- fallback: scroll + throttle ----------
    const handler = throttle(() => {
      const rect = el.getBoundingClientRect()
      if (rect.top <= window.innerHeight + 150) {
        tryLoad()
      }
    }, 200)

    window.addEventListener('scroll', handler, { passive: true })
    handler()

    return () => {
      window.removeEventListener('scroll', handler)
    }
  }, [sentinelRef, disabled, noMore, root, rootMargin, threshold])
}

```

