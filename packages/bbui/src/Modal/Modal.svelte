<script>
  import "@spectrum-css/modal/dist/index-vars.css"
  import "@spectrum-css/underlay/dist/index-vars.css"
  import { createEventDispatcher, setContext, tick } from "svelte"
  import { fade, fly } from "svelte/transition"
  import Portal from "svelte-portal"
  import Context from "../context"

  export let fixed = false
  export let inline = false

  const dispatch = createEventDispatcher()
  let visible = fixed || inline
  $: dispatch(visible ? "show" : "hide")

  export function show() {
    if (visible) {
      return
    }
    visible = true
  }

  export function hide() {
    if (!visible || fixed || inline) {
      return
    }
    visible = false
  }

  export function cancel() {
    if (!visible) {
      return
    }
    dispatch("cancel")
    hide()
  }

  function handleKey(e) {
    if (visible && e.key === "Escape") {
      cancel()
    }
  }

  async function focusFirstInput(node) {
    const inputs = node.querySelectorAll("input")
    if (inputs?.length) {
      await tick()
      inputs[0].focus()
    }
  }

  setContext(Context.Modal, { show, hide, cancel })
</script>

<svelte:window on:keydown={handleKey} />

{#if inline}
  {#if visible}
    <div use:focusFirstInput class="spectrum-Modal inline is-open">
      <slot />
    </div>
  {/if}
{:else}
  <!--
    We cannot conditionally render the portal as this leads to a missing
    insertion point when using nested modals. Therefore we just conditionally
    render the content of the portal.
    It still breaks the modal animation, but its better than soft bricking the
    screen.
  -->
  <Portal target=".modal-container">
    {#if visible}
      <div
        class="spectrum-Underlay is-open"
        in:fade={{ duration: 200 }}
        out:fade|local={{ duration: 200 }}
        on:mousedown|self={cancel}
      >
        <div class="modal-wrapper" on:mousedown|self={cancel}>
          <div class="modal-inner-wrapper" on:mousedown|self={cancel}>
            <slot name="outside" />
            <div
              use:focusFirstInput
              class="spectrum-Modal is-open"
              in:fly={{ y: 30, duration: 200 }}
              out:fly|local={{ y: 30, duration: 200 }}
            >
              <slot />
            </div>
          </div>
        </div>
      </div>
    {/if}
  </Portal>
{/if}

<style>
  .spectrum-Underlay {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    z-index: 999;
    overflow: auto;
    overflow-x: hidden;
    background: rgba(0, 0, 0, 0.75);
  }

  .modal-wrapper {
    flex: 1 1 auto;
    display: flex;
    flex-direction: row;
    -moz-box-pack: center;
    justify-content: center;
    align-items: flex-start;
    max-height: 100%;
  }
  .modal-inner-wrapper {
    flex: 1 1 auto;
    display: flex;
    flex-direction: row;
    -moz-box-pack: center;
    justify-content: center;
    align-items: flex-start;
    width: 0;
    position: relative;
  }

  .spectrum-Modal {
    overflow: visible;
    max-height: none;
    margin: 40px 0;
    transform: none;
    --spectrum-dialog-confirm-border-radius: var(
      --spectrum-global-dimension-size-100
    );
    max-width: 100%;
  }
  :global(.spectrum--lightest .spectrum-Modal.inline) {
    border: var(--border-light);
  }
</style>
