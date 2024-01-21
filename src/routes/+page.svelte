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
      title: "Identité du bot",
      description: "Quel est le nom du bot ?",
      name: "botName",
      type: "text",
      category: "identity",
      default: "Bachero"
    },
    {
      title: "Couleur d'embed",
      description: "Quelle couleur sera principalement utilisée dans les embeds ?",
      name: "embedColor",
      type: "color",
      category: "color",
      default: "#6A82FB"
    },
    {
      type: "skipper"
    },
    {
      title: "Couleur d'embed",
      description: "Quelle couleur d'accentuation sera utilisée dans les embeds ?",
      name: "secondEmbedColor",
      type: "color",
      category: "color",
      default: "#EAB464"
    },
    {
      type: "skipper"
    },
    {
      title: "Couleur d'embed",
      description: "Quelle couleur doit être utilisée pour indiquer un accomplissement dans un embed ?",
      name: "successEmbedColor",
      type: "color",
      category: "color",
      default: "#3BA55C"
    },
    {
      type: "skipper",
    },
    {
      title: "Couleur d'embed",
      description: "Quelle couleur doit être utilisée pour indiquer une erreur d'exécution dans un embed ?",
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
          label: "MongoDB (à configurer via .env)",
          value: "mongodb"
        }
      ],
      required: true
    },
    {
      title: "Statut du bot",
      description: "Quel statut d'activité voulez-vous avoir pour le bot ?",
      name: "botActivityContent",
      type: "text_not_required",
      category: "activity",
      default: "bachero.johanstick.fr",
    },
    {
      title: "Statut du bot",
      description: "Quel type de statut d'activité voulez-vous définir ?",
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
          label: "Participant à",
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
      title: "Commandes textes",
      description: "Quel préfixe voulez-vous utiliser pour les commandes textuelles ?",
      name: "prefix",
      type: "text",
      category: "commands",
      default: "bc "
    },
    {
      title: "Commandes textes",
      description: "Voulez-vous désactiver les commandes textes ? Si désactivées, seules les commandes slash seront utilisables.",
      name: "disableTextCommand",
      type: "yes_or_no",
      category: "commands",
    },
    {
      title: "Utilisateurs autorisés",
      description: "Voulez-vous autoriser les bots à utiliser les commandes textes de Bachero ?",
      name: "letBotUseCommands",
      type: "yes_or_no",
      category: "commands"
    },
    {
      title: "Vérification des mises à jour",
      description: "Voulez-vous désactiver la vérification des mises à jour du bot au démarrage ?",
      name: "disableCheckUpdate",
      type: "yes_or_no",
      category: "debugging"
    },
    {
      title: "Rapports d'erreurs",
      description: "Voulez-vous désactiver la création de rapports en cas d'erreurs ?",
      name: "disableReport",
      type: "yes_or_no",
      category: "debugging"
    },
    {
      title: "Logs",
      description: "Voulez-vous conserver les logs dans des fichiers ?",
      name: "outputLogsInFile",
      type: "yes_or_no",
      category: "debugging"
    },
    {
      title: "Logs",
      description: "Voulez-vous afficher les logs de débogage dans la console ? Cela peut être utile pendant le développement de modules",
      name: "showDebugLogsInConsole",
      type: "yes_or_no",
      category: "debugging"
    },
    {
      title: "Minification des fichiers générés",
      description: "Voulez-vous désactiver la minification des fichiers générés pour les commandes textes ? Cela rend le code plus simple à lire",
      name: "disableMinifyingTextCmdsFiles",
      type: "yes_or_no",
      category: "debugging"
    },
    {
      title: "Analytics",
      description: "Voulez-vous désactiver le suivi de l'utilisation des commandes ? Aucune donnée n'est envoyée à Bachero, indispensable pour la commande /analytics",
      name: "disableCommandAnalytics",
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
    let d = await fetch("https://haste.johanstick.fr/documents", {
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

<div class="content dark flex items-center absolute inset-0 justify-center p-4">
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

<svelte:head>
  <title>Bachero Utilities</title>
  <meta name="description" content="Un simple configurateur visuel pour Bachero" />
  <meta name="keywords" content="bachero, utilities, bot, discord, discord.js, discord bot, bot discord, bot discord.js, discord.js bot" />
  <meta name="author" content="Bachero" />
  <meta name="robots" content="index, nofollow" />

  <meta property="og:title" content="Bachero Utilities" />
  <meta property="og:description" content="Un simple configurateur visuel pour Bachero" />
  <meta property="og:type" content="configurator" />
  <meta property="og:url" content="https://config.bachero.johanstick.fr" />
  <meta property="og:image" content="https://bachero.johanstick.fr/icon/carre.png" />
  <meta property="og:site_name" content="Bachero Utilities" />
  <meta property="og:locale" content="fr_FR" />

  <meta name="twitter:title" content="Bachero Utilities" />
  <meta name="twitter:description" content="Un simple configurateur visuel pour Bachero" />
  <meta name="twitter:card" content="summary" />
  <meta name="twitter:image" content="https://bachero.johanstick.fr/icon/carre.png" />
</svelte:head>