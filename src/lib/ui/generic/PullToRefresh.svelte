<script lang="ts">
  import { Spinner } from 'mono-svelte'
  import { ArrowPath, Icon } from 'svelte-hero-icons/dist'
  import { scrollY } from 'svelte/reactivity/window'

  interface Props {
    onrefresh: () => Promise<void>
    /** How far the user must pull (px) before release triggers a refresh. */
    threshold?: number
  }

  let { onrefresh, threshold = 80 }: Props = $props()

  // Half the indicator's height (40px tall → 20px).
  const HALF = 20

  let startY = $state(0)
  let pullY = $state(0)
  let pulling = $state(false)
  let refreshing = $state(false)

  /**
   * Visual pull distance — eased so it slows down past the threshold, and
   * stays put while refreshing.
   */
  const visualPull = $derived(
    refreshing
      ? threshold * 0.55       // anchor indicator while loading
      : Math.min(pullY, threshold * 1.5) * 0.5, // 2:1 ratio, capped
  )

  /** 0→1 representing how far through the threshold the user has pulled. */
  const progress = $derived(Math.min(pullY / threshold, 1))

  /** Opacity fades in over the first 20px of visual travel. */
  const opacity = $derived(Math.min(visualPull / HALF, 1))

  /** Vertical offset of the indicator's center from the top of the screen. */
  const translateY = $derived(visualPull - HALF)

  /**
   * Apply a CSS transition only when the finger is NOT moving so that the
   * indicator tracks the touch instantly but animates back smoothly.
   */
  const cssTransition = $derived(
    pulling
      ? 'none'
      : 'transform 0.35s cubic-bezier(0.075, 0.82, 0.165, 1), opacity 0.35s ease',
  )

  function ontouchstart(e: TouchEvent) {
    if (refreshing || (scrollY.current ?? 0) > 2) return
    startY = e.touches[0].clientY
    pulling = true
  }

  function ontouchmove(e: TouchEvent) {
    if (!pulling || refreshing) return
    // If the user has scrolled away from the very top, cancel.
    if ((scrollY.current ?? 0) > 2) {
      pulling = false
      pullY = 0
      return
    }
    pullY = Math.max(0, e.touches[0].clientY - startY)
  }

  async function ontouchend() {
    if (!pulling) return
    pulling = false

    if (pullY >= threshold && !refreshing) {
      pullY = 0
      refreshing = true
      try {
        await onrefresh()
      } finally {
        refreshing = false
      }
    } else {
      pullY = 0
    }
  }
</script>

<svelte:window {ontouchstart} {ontouchmove} {ontouchend} />

<!--
  Always in the DOM (opacity:0 when idle) so CSS transitions play correctly.
  pointer-events:none keeps it from blocking any taps.
-->
<div
  role="status"
  aria-label={refreshing ? 'Refreshing…' : 'Pull down to refresh'}
  aria-live="polite"
  class="ptr-wrap"
  style="
    opacity: {opacity};
    transform: translateX(-50%) translateY({translateY}px);
    transition: {cssTransition};
  "
>
  {#if refreshing}
    <Spinner width={18} />
  {:else}
    <div style="transform: rotate({progress * 270}deg);">
      <Icon src={ArrowPath} size="18" />
    </div>
  {/if}
</div>

<style>
  .ptr-wrap {
    position: fixed;
    top: 0;
    left: 50%;
    z-index: 200;
    display: flex;
    align-items: center;
    justify-content: center;
    width: 40px;
    height: 40px;
    border-radius: 9999px;
    /* match the app's card/sidebar surface */
    background-color: var(--color-slate-50, #f8fafc);
    border: 1px solid var(--color-slate-200, #e2e8f0);
    box-shadow: 0 2px 10px rgb(0 0 0 / 0.12);
    color: var(--color-slate-600, #475569);
    /* prevent any accidental pointer interference */
    pointer-events: none;
    /* GPU layer hint so the transform is cheap */
    will-change: transform, opacity;
  }

  :global(.dark) .ptr-wrap {
    background-color: var(--color-zinc-950, #09090b);
    border-color: var(--color-zinc-800, #27272a);
    color: var(--color-zinc-400, #a1a1aa);
    box-shadow: 0 2px 10px rgb(0 0 0 / 0.35);
  }
</style>
