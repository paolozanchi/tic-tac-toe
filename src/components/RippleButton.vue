<template>
  <button
    :disabled="disabled"
    :variant="variant"
    @click="createRipple"
  >
    <slot />
  </button>
</template>

<script>
export default {
  name: 'RippleButton',
  props: {
    disabled: Boolean,
    variant: {
      type: String,
      default: 'light'
    },
  },
  methods: {
    createRipple(event) {
      const button = event.currentTarget;
      const circle = document.createElement("span");
      const diameter = Math.max(button.clientWidth, button.clientHeight);
      const radius = diameter / 2;

      circle.style.width = circle.style.height = `${diameter}px`;
      circle.style.left = `${event.clientX - (button.offsetLeft + radius)}px`;
      circle.style.top = `${event.clientY - (button.offsetTop + radius)}px`;
      circle.classList.add("ripple"); 

      const ripple = button.getElementsByClassName("ripple")[0];

      if (ripple) {
        ripple.remove();
      }

      button.appendChild(circle);

      this.$emit('click');
    }
  }
}
</script>

<style scoped>
  button {
    position: relative;
    overflow: hidden;
    transition: background 400ms;
    color: var(--dark);
    background-color: var(--light, #eeeeee);
    margin: 0.5rem;
    padding: 0.65rem;
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Helvetica Neue", Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
    font-size: 1rem;
    outline: 0;
    border: 0;
    border-radius: var(--border-radius, .25rem);
    cursor: pointer;
  }
  
  button[variant="primary"] {
    background-color: var(--accent);
  }
  
  button[variant="light"] {
    background-color: var(--light);
    color: var(--dark);
  }
  
  button[variant="dark"] {
    background-color: var(--dark);
    color: var(--light);
  }
  
  button[variant="success"] {
    background-color: var(--success);
    color: var(--light);
  }
  
  button[variant="danger"] {
    background-color: var(--danger);
    color: var(--light);
  }
  
  button:disabled {
    opacity: .65;
    cursor: not-allowed;
  }
</style>

<style>
  span.ripple {
    position: absolute;
    border-radius: 50%;
    transform: scale(0);
    animation: ripple 600ms linear;
    background-color: rgba(var(--rippleRgb), 0.6);
  }
  
  @keyframes ripple {
    to {
      transform: scale(4);
      opacity: 0;
    }
  }
</style>