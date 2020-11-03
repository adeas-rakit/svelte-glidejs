<!-- your code here -->
<style src="../node_modules/@glidejs/glide/dist/css/glide.core.css"></style>
<script>
    import Glide from '@glidejs/glide'
    import { createEventDispatcher } from 'svelte';

    const dispatch = createEventDispatcher();

    export let items=[];
    export let options={};
    export let bullet=false;
    export let control=false;

    let glide;
    let controller;
    let active;

    function initGlide(node, options) {
      const events = [
              'mount.before', 'run', 'mount.after', 'update', 'play', 'pause',
              'build.before', 'build.after', 'run.before', 'run.after',
              'run.offset', 'run.start', 'run.end', 'move', 'move.after',
              'resize', 'swipe.start', 'swipe.move', 'swipe.end', 'translate.jump',
      ];

      glide = new Glide(node, options);

      // forward glide event
      events.forEach(function (event) {
          glide.on(event, function(args) {
              glide = glide;  // reactive
              // Replace event name from a.b to aB or keep source
              dispatch(event.replace(/\.\w/, (v) => v[1].toUpperCase()), args)
          });
      });

      glide.mount();

      return {
          update: glide.update,
          //destroy: glide.disable
      }
    }

    function bulletIn(index, glide){
      return {
          focus: () => glide.go(`=${index}`),
          isActive:  glide.index === index
      }
    }
</script>
<div use:initGlide={options} class="glide">
  <div class="glide__track" data-glide-el="track">
      <ul class="glide__slides">
        {#each items as item, index}
            <li class="glide__slide">
                <slot name="item" {item} {glide} {index}>{index}</slot>
            </li>
        {/each}
      </ul>
  </div>
  {#if control && glide}
    <div class="glide__arrows" >
        <slot name="control" {glide}>
            <button on:click={() => glide.go('<')} class="glide__arrow glide__arrow--left" aria-label="Left Arrow">
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24"><path d="M0 12l10.975 11 2.848-2.828-6.176-6.176H24v-3.992H7.646l6.176-6.176L10.975 1 0 12z"></path></svg>
            </button>
            <button on:click={() => glide.go('>')} class="glide__arrow glide__arrow--right" aria-label="Right Arrow">
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24"><path d="M13.025 1l-2.847 2.828 6.176 6.176h-16.354v3.992h16.354l-6.176 6.176 2.847 2.828 10.975-11z"></path></svg>
            </button>
        </slot>
    </div>
  {/if}
  {#if bullet && glide }
    <div class="glide__bullets">
      {#each items as item, index}
          <slot name="bullet" prop={bulletIn(index, glide)}>
              <button  aria-label="Bullet-{index}" class:focus={bulletIn(index, glide).isActive}
                      class="glide__bullet"
                      on:click={() => bulletIn(index, glide).focus()}
              >

              </button>
          </slot>
      {/each}
    </div>
  {/if}
</div>
