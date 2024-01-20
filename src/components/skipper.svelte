<script lang="ts">
	export let step: number;
  export let lastStep: number;

  if (Math.sign(step - lastStep) === -1) {
    step--;
  } else {
    step++;
  }
</script>