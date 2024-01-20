<script lang="ts">
  import * as Card from "$lib/components/ui/card";
  import {Check, Cross1, ArrowLeft, Avatar, EyeOpen, Pencil1, Cube, Slash, Gear} from "radix-icons-svelte";
  import {Button} from "$lib/components/ui/button";

  export let step: number;
  export let responses: { [key: string]: string | boolean };
  export let name: string;
  export let title: string;
  export let description: string;
  export let category: string;
  export let callback: (res: boolean) => void;
  export let howTo: string;
  export let howToLabel: string;
  export let lastStep: number;

  function onSubmit(s: boolean) {
    responses[name] = s;
    if (callback) callback(s);
    lastStep = step;
    step++;
  }

  function onReturn() {
    lastStep = step;
    step--;
  }
</script>

<Card.Root>
  <Card.Header>
    <Card.Title class="flex">
      {#if category === "identity"}
        <Avatar class="mr-2 h-5 w-5" />
      {:else if category === "color"}
        <Pencil1 class="mr-2 h-5 w-5" />
      {:else if category === "activity"}
        <EyeOpen class="mr-2 h-5 w-5" />
      {:else if category === "database"}
        <Cube class="mr-2 h-5 w-5" />
      {:else if category === "commands"}
        <Slash class="mr-2 h-5 w-5" />
      {:else if category === "debugging"}
        <Gear class="mr-2 h-5 w-5" />
      {/if} {title}</Card.Title>
    <Card.Description>{description}</Card.Description>
  </Card.Header>
  <Card.Content>
    {#if howTo}
      <a href={howTo} class="hover:underline text-xs" target="_blank">{howToLabel}</a>
    {/if}
  </Card.Content>
  <Card.Footer>
    <Button class="w-full" on:click={onReturn}>
      <ArrowLeft class="mr-2 h-4 w-4" /> Retour
    </Button>
    <Button class="w-full ml-2" on:click={() => { onSubmit(true) }}>
      <Check class="mr-2 h-4 w-4" /> Oui
    </Button>
    <Button class="w-full ml-2" on:click={() => { onSubmit(false) }}>
      <Cross1 class="mr-2 h-4 w-4" /> Non
    </Button>
  </Card.Footer>
</Card.Root>