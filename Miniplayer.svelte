<script>
  export let active = false
  export let padding = '10px'
  export let maxwidth = '100%'
  export let minwidth = '0%'
  export let resize = true
  export let width = resize ? '400px' : 'auto'
  let height = '0px'
  let left = '0px'
  let top = '0px'
  export let position = 'bottom right'
  let container = null
  let dragging = false
  function draggable (node) {
    const initial = { x: 0, y: 0 }
    let timeout = null
    let tmppadding = null
    function dragStart (e) {
      clearTimeout(timeout)
      tmppadding = padding
      padding = '0px'
      const { pointerId, offsetX, offsetY } = e
      const bounds = container.getBoundingClientRect()
      dragging = true
      position = ''
      initial.x = offsetX
      initial.y = bounds.height - offsetY
      width = bounds.width + 'px'
      height = bounds.height + 'px'
      handleDrag(e)
      document.body.addEventListener('pointermove', handleDrag)
      if (pointerId) node.setPointerCapture(pointerId)
    }
    function dragEnd ({ pointerId, clientX, clientY }) {
      dragging = false
      padding = tmppadding
      document.body.removeEventListener('pointermove', handleDrag)
      const istop = window.innerHeight / 2 - clientY >= 0
      const isleft = window.innerWidth / 2 - clientX >= 0
      top = istop ? '0px' : '100%'
      left = isleft ? '0px' : '100%'
      if (pointerId) node.releasePointerCapture(pointerId)
      timeout = setTimeout(() => {
        position += istop ? ' top' : ' bottom'
        position += isleft ? ' left' : ' right'
      }, 600)
    }
    function handleDrag ({ clientX, clientY }) {
      left = clientX - initial.x + 'px'
      top = clientY - initial.y + 'px'
    }
    node.addEventListener('pointerdown', dragStart)
    node.addEventListener('pointerup', dragEnd)
    node.addEventListener('touchstart', dragStart)
    node.addEventListener('touchend', dragEnd)
  }

  function resizable (node) {
    function resizeStart ({ pointerId }) {
      document.body.addEventListener('pointermove', handleResize)
      if (pointerId) node.setPointerCapture(pointerId)
    }
    function resizeEnd ({ pointerId }) {
      document.body.removeEventListener('pointermove', handleResize)
      if (pointerId) node.setPointerCapture(pointerId)
    }
    function handleResize ({ movementX }) {
      width = width.slice(0, -2) - movementX + 'px'
    }
    node.addEventListener('pointerdown', resizeStart)
    node.addEventListener('pointerup', resizeEnd)
    node.addEventListener('touchstart', resizeStart)
    node.addEventListener('touchend', resizeEnd)
  }
</script>
<div class='miniplayer-container {position} {$$restProps.class}'
  class:active class:animate={!dragging} class:custompos={!position}
  style:--left={left} style:--top={top} style:--height={height} style:--width={width} style:--padding={padding} style:--maxwidth={maxwidth} style:--minwidth={minwidth}
  bind:this={container} on:dragstart|preventDefault>
  {#if resize && active}
    <div class='resize' use:resizable></div>
  {/if}
  <slot />
  <div class='miniplayer-footer' class:dragging use:draggable>::::</div>
</div>

<style>
  .resize {
    background: linear-gradient(to bottom right, rgba(0, 0, 0, 0.5) 00%, rgba(0, 0, 0, 0.5) 50%, transparent 50%, transparent 100%);
    position: absolute;
    top: 0;
    left: 0;
    cursor: nw-resize;
    user-select: none;
    width: 15px;
    height: 15px;
    z-index: 100;
  }
  .active {
    position: absolute;
    width: clamp(var(--minwidth), var(--width), var(--maxwidth)) !important
  }
  .active.custompos {
    top: clamp(var(--padding), var(--top), 100% - var(--height) - var(--padding)) !important;
    left: clamp(var(--padding), var(--left), 100% - var(--width) - var(--padding)) !important;
  }
  .active.top {
    top: var(--padding) !important
  }
  .active.bottom {
    bottom: var(--padding) !important
  }
  .active.left {
    left: var(--padding) !important
  }
  .active.right {
    right: var(--padding) !important
  }
  .animate {
    transition-duration: 0.5s;
    transition-property: top, left;
    transition-timing-function: cubic-bezier(0.3, 1.5, 0.8, 1);
  }
  .miniplayer-footer {
    display: none;
    letter-spacing: 1.5px;
    cursor: grab;
    font-weight: 600;
    user-select: none;
    padding-bottom: 0.2rem;
    text-align: center;
  }
  .dragging {
    cursor: grabbing !important;
  }
  .active > .miniplayer-footer {
    display: block !important;
  }
</style>
