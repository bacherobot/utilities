<script lang="ts">
  import Text from "../components/text.svelte";
  import TextNotRequired from "../components/text_not_required.svelte";
  import Color from "../components/color.svelte";
  import Skipper from "../components/skipper.svelte";
  import Multichoice from "../components/multiple_choices.svelte";
  import YesOrNo from "../components/yes_or_no.svelte";
  import * as Card from "$lib/components/ui/card";
  import { Button } from "$lib/components/ui/button";
  import {Download} from "radix-icons-svelte";
  import JSONC from 'tiny-jsonc';
  import { Skeleton } from "$lib/components/ui/skeleton";

  const bacheroVersion = "0.12.0-beta";
  let step = 0;
  let lastStep = 0;
  let model = "https://raw.githubusercontent.com/bacherobot/bot/master/config/bachero.jsonc";
  let steps = [
    {
      title: "Nom du bot",
      description: "Quel est le nom du bot ?",
      name: "botName",
      type: "text",
      category: "identity",
      default: "Bachero"
    },
    {
      title: "Couleur principale de l'embed",
      description: "La couleur utilisée principalement pour les embeds",
      name: "embedColor",
      type: "color",
      category: "color",
      default: "#6A82FB"
    },
    {
      type: "skipper"
    },
    {
      title: "Couleur secondaire de l'embed",
      description: "La couleur utilisée parfois pour les embeds",
      name: "secondEmbedColor",
      type: "color",
      category: "color",
      default: "#EAB464"
    },
    {
      type: "skipper"
    },
    {
      title: "Couleur de succès de l'embed",
      description: "La couleur utilisée pour indiquer le succès d'une commande via un embed",
      name: "successEmbedColor",
      type: "color",
      category: "color",
      default: "#3BA55C"
    },
    {
      type: "skipper",
    },
    {
      title: "Couleur d'erreur de l'embed",
      description: "La couleur utilisée pour indiquer l'erreur d'une commande via un embed",
      name: "dangerEmbedColor",
      type: "color",
      category: "color",
      default: "#ED4245"
    },
    {
      title: "Base de données",
      description: "Quel type de base de données voulez-vous utiliser ?",
      name: "databaseType",
      type: "multichoice",
      category: "database",
      placeholder: "Choisissez une base de données",
      choices: [
        {
          label: "JSON",
          value: "json"
        },
        {
          label: "MongoDB",
          value: "mongodb"
        }
      ],
      required: true
    },
    {
      title: "Statut d'activité du bot",
      description: "Quel statut d'activité voulez-vous avoir pour le bot ?",
      name: "botActivityContent",
      type: "text_not_required",
      category: "activity",
      default: "bachero.johanstick.fr",
    },
    {
      title: "Type de statut d'activité du bot",
      description: "Quel type de statut d'activité voulez-vous avoir pour le bot ?",
      name: "botActivityType",
      type: "multichoice",
      category: "activity",
      placeholder: "Choisissez un type de statut d'activité",
      choices: [
        {
          label: "Joue à",
          value: "playing"
        },
        {
          label: "Stream",
          value: "streaming"
        },
        {
          label: "Regarde",
          value: "watching"
        },
        {
          label: "Écoute",
          value: "listening"
        },
        {
          label: "Compétition",
          value: "competing"
        },
        {
          label: "Personnalisé",
          value: "custom"
        }
      ],
      required: false
    },
    {
      type: "skipper"
    },
    {
      title: "Statut du bot",
      description: "Quel statut voulez-vous avoir pour le bot ?",
      name: "botStatus",
      type: "multichoice",
      category: "activity",
      placeholder: "Choisissez un statut",
      choices: [
        {
          label: "En ligne",
          value: "online"
        },
        {
          label: "Inactif",
          value: "idle"
        },
        {
          label: "Ne pas déranger",
          value: "dnd"
        },
        {
          label: "Invisible",
          value: "offline"
        }
      ],
      required: true
    },
    {
      title: "Préfixe des commandes",
      description: "Quel préfixe voulez-vous utiliser pour les commandes ?",
      name: "prefix",
      type: "text",
      category: "commands",
      default: "bc "
    },
    {
      title: "Commandes textes",
      description: "Voulez-vous désactivé les commandes textes ? Si désactivé, seul les commandes slash seront utilisables.",
      name: "disableTextCommand",
      type: "yes_or_no",
      category: "commands",
    },
    {
      title: "Utilisateur autorisés",
      description: "Voulez-vous autorisé les bots à utiliser les commandes textes de Bachero ?",
      name: "letBotUseCommands",
      type: "yes_or_no",
      category: "commands"
    },
    {
      title: "Mise à jour automatique",
      description: "Voulez-vous désactivé la vérification automatique de mise à jour ?",
      name: "disableCheckUpdate",
      type: "yes_or_no",
      category: "debugging"
    },
    {
      title: "Rapports d'erreurs",
      description: "Voulez-vous désactivé les rapports d'erreurs ?",
      name: "disableReport",
      type: "yes_or_no",
      category: "debugging"
    },
    {
      title: "Logs",
      description: "Voulez-vous activé les fichiers de logs ?",
      name: "outputLogsInFile",
      type: "yes_or_no",
      category: "debugging"
    },
    {
      title: "Minification des fichiers générés",
      description: "Voulez-vous désactivé la minification des fichiers générés pour les commandes textes ?",
      name: "minifyGeneratedFiles",
      type: "yes_or_no",
      category: "debugging"
    },
    {
      title: "Analytics",
      description: "Voulez-vous désactivé les analytics ?",
      name: "disableCommandAnalytics",
      type: "yes_or_no",
      category: "debugging"
    },
    {
      title: "Logs",
      description: "Voulez-vous affiché les logs de debugging dans la console ?",
      name: "showDebugLogsInConsole",
      type: "yes_or_no",
      category: "debugging"
    }
  ]
  let responses = {};

  async function generateJson() {
    let d = await fetch(model)
      .then(res => res.text())

    let data = JSONC.parse(d);
    for (let i = 0; i < data.config.length; i++) {
      let c = data.config[i];
      for (let i = 0; i < Object.keys(responses).length; i++) {
        let r = Object.keys(responses)[i];
        if (c.name === r) {
          c.value = responses[r];
        }
      }
    }

    return `/*\n * Generated by Bachero Utilities\n * For Bachero v${bacheroVersion}\n * ${new Date().toISOString().split("T")[0]}\n * ${new Date().toISOString().split("T")[1].split(".")[0]}\n*/\n\n` + JSON.stringify(data, null, 2);
  }

  function downloadJson() {
    generateJson().then(json => {
      let blob = new Blob([json], { type: "application/json" });
      let url = URL.createObjectURL(blob);
      let a = document.createElement("a");
      a.href = url;
      a.download = "bachero.jsonc";
      a.click();
    })
  }

  async function generateHastebin() {
    let json = await generateJson();
    let d = await fetch("https:/haste.johanstick.fr/documents", {
      method: "POST",
      body: json
    }).then(res => res.json());

    return `https://haste.johanstick.fr/raw/${d.key}.json`;
  }

  async function hastebinCommand() {
    let link = await generateHastebin();
    return `curl -o bachero.jsonc ${link}`;
  }
</script>

<div class="content dark flex items-center w-screen h-screen justify-center">
  {#if steps[step]}
    {#if steps[step].type === "text"}
      <Text
          bind:step
          bind:lastStep
          bind:responses
          name={steps[step].name}
          title={steps[step].title}
          description={steps[step].description}
          category={steps[step].category}
          callback={steps[step].callback}
          howTo={steps[step].howTo}
          howToLabel={steps[step].howToLabel}
          Default={steps[step].default}
      />
    {:else if steps[step].type === "color"}
      <Color
          bind:step
          bind:lastStep
          bind:responses
          name={steps[step].name}
          title={steps[step].title}
          description={steps[step].description}
          category={steps[step].category}
          callback={steps[step].callback}
          howTo={steps[step].howTo}
          howToLabel={steps[step].howToLabel}
          Default={steps[step].default}
      />
    {:else if steps[step].type === "skipper"}
      <Skipper bind:step bind:lastStep />
    {:else if steps[step].type === "multichoice"}
      <Multichoice
          bind:step
          bind:lastStep
          bind:responses
          name={steps[step].name}
          title={steps[step].title}
          description={steps[step].description}
          category={steps[step].category}
          callback={steps[step].callback}
          howTo={steps[step].howTo}
          howToLabel={steps[step].howToLabel}
          placeholder={steps[step].placeholder}
          choices={steps[step].choices}
          required={steps[step].required}
      />
    {:else if steps[step].type === "text_not_required"}
      <TextNotRequired
          bind:step
          bind:lastStep
          bind:responses
          name={steps[step].name}
          title={steps[step].title}
          description={steps[step].description}
          category={steps[step].category}
          callback={steps[step].callback}
          howTo={steps[step].howTo}
          howToLabel={steps[step].howToLabel}
          Default={steps[step].default}
      />
    {:else if steps[step].type === "yes_or_no"}
      <YesOrNo
          bind:step
          bind:lastStep
          bind:responses
          name={steps[step].name}
          title={steps[step].title}
          description={steps[step].description}
          category={steps[step].category}
          callback={steps[step].callback}
          howTo={steps[step].howTo}
          howToLabel={steps[step].howToLabel}
      />
    {/if}
  {:else}
    <Card.Root class="max-[450px]:w-full">
      <Card.Header>
        <Card.Title>Configuration terminée</Card.Title>
        <Card.Description>Vous pouvez maintenant télécharger votre configuration.</Card.Description>
      </Card.Header>
      <Card.Content>
        <code class="break-words">
          {#await hastebinCommand()}
            <Skeleton class="w-full" />
          {:then command}
            {command}
          {:catch}
            <Skeleton class="w-full" />
          {/await}
        </code>
      </Card.Content>
      <Card.Footer>
        <Button class="w-full" on:click={downloadJson}>
          <Download class="mr-2 h-4 w-4" /> Télécharger directement le contenu
        </Button>
      </Card.Footer>
    </Card.Root>
  {/if}
</div>

<style lang="postcss">
  .dark {
      --cp-bg-color: rgba(0, 0, 0, 0);
  }
</style>