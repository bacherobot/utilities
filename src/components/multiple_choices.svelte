<script lang="ts">
  import * as Card from "$lib/components/ui/card";
  import * as Select from "$lib/components/ui/select";
  import {ArrowRight, ArrowLeft, Avatar, EyeOpen, Pencil1, Cube, Slash, Gear} from "radix-icons-svelte";
  import {Button} from "$lib/components/ui/button";

  export let step: number;
  export let responses: { [key: string]: string };
  export let name: string;
  export let title: string;
  export let description: string;
  export let category: string;
  export let callback: (res: string) => void;
  export let howTo: string;
  export let howToLabel: string;
  export let lastStep: number;
  export let placeholder: string;
  export let required: boolean;

  interface Choice {
    label: string;
    value: string;
  }
  export let choices: Choice[];

  function onSubmit() {
    if (required) if (!(document.getElementById("response") as HTMLInputElement).value) return;
    if (!required && !(document.getElementById("response") as HTMLInputElement).value) {
      lastStep = step;
      step++;
      return;
    }
    responses[name] = (document.getElementById("response") as HTMLInputElement).value;
    if (callback) callback((document.getElementById("response") as HTMLInputElement).value);
    lastStep = step;
    step++;
  }

  function onReturn() {
    lastStep = step;
    step--;
  }
</script>

<Card.Root class="w-full md:w-auto">
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
    <Select.Root >
      <Select.Trigger class="w-full md:w-[300px]">
        <Select.Value placeholder={placeholder} />
      </Select.Trigger>
      <Select.Content>
        <Select.Group>
          {#each choices as choice}
            <Select.Item value={choice.value} label={choice.label}>{choice.label}</Select.Item>
          {/each}
        </Select.Group>
      </Select.Content>
      <Select.Input name="response" id="response" />
    </Select.Root>
    {#if howTo}
      <a href={howTo} class="hover:underline text-xs" target="_blank">{howToLabel}</a>
    {/if}
  </Card.Content>
  <Card.Footer>
    <Button class="w-full" on:click={onReturn}>
      <ArrowLeft class="mr-2 h-4 w-4" /> Retour
    </Button>

    <Button class="w-full ml-2" on:click={onSubmit}>
      <ArrowRight class="mr-2 h-4 w-4" /> Suivant
    </Button>
  </Card.Footer>
</Card.Root>