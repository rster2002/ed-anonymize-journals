<main>
  <section>
    <h1>Anonymize journals</h1>

    <p>
      Replaces identifying information like CMDR name and Frontier ID with generic values so you can contribute your
      journal files to the project.
    </p>

    <p>
      Things that will be anonymized:
    </p>

    <ol>
      <li>Your commander name</li>
      <li>Your Frontier ID</li>
      <li>The names of your friends in friend-related events</li>
      <li>Contents of sent and received messages to and from players and their names</li>
      <li>CMDR names from wing related events</li>
      <li>CMDR names from multi-crew related events</li>
    </ol>

    <p>
      Note that this website will NOT anonymize the things like which systems you've visited and your flight log because
      it's very difficult to determine which names to remove and which to keep. Because your commander name is anonymize
      it shouldn't be an issue, but it's important to mention.
    </p>

    {#if !working}
      <input bind:value={commanderName} type="text" placeholder="CMDR name (without CMDR prefix)" />

      <input bind:this={fileEl} type="file" multiple />

      {#if errorMessage.length > 0}
        <p>
          {errorMessage}
        </p>
      {/if}

      <button on:click={performAnonymization}>
        Anonymize
      </button>
    {:else}
      <p>
        Working...
      </p>
    {/if}
  </section>
</main>

<script lang="ts">
import JSZip from "jszip";

let commanderName = "";
let working = false;
let fileEl: HTMLInputElement;
let errorMessage = "";

async function performAnonymization() {
  working = true;

  try {
    const processedFiles = await Promise.all(
      Array.from(fileEl.files).map(file => {
        return (async () => ({
          fileName: file.name,
          contents: await processFile(file),
        }))();
      })
    );

    const zip = new JSZip();

    for (const file of processedFiles) {
      zip.file(file.fileName, file.contents);
    }

    zip.generateAsync({type:'blob'}).then((blobdata)=>{
      // create zip blob file
      let zipblob = new Blob([blobdata])

      // For development and testing purpose
      // Download the zipped file
      let elem = window.document.createElement("a")
      elem.href = window.URL.createObjectURL(zipblob)
      elem.download = 'compressed.zip'
      elem.click()
    });
  } catch (e) {
    errorMessage = e.toString();
  } finally {
    working = false;
  }
}

async function processFile(file: File) {
  const contents = await file.text();

  return contents.split("\n")
    .filter(line => line.trim().length > 0 && !line.includes("\0"))
    .map(line => line.split(commanderName.toUpperCase()).join("SOMECOMMANDER"))
    .map(line => JSON.parse(line.trim()))
    .map(line => {
      if (line.event === "Commander") {
        return {
          ...line,
          FID: "F123456789",
          Name: "TEST",
        };
      }

      if (line.event === "LoadGame") {
        return {
          ...line,
          FID: "F123456789",
          Commander: "TEST",
        };
      }

      if (line.event === "Friends") {
        return {
          ...line,
          Name: "TEST FRIEND",
        };
      }

      if (line.event === "SendText") {
        return {
          ...line,
          To: "SOME RECIPIENT",
          Message: "Some awesome message that is just to cool to show."
        };
      }

      if (line.event === "ReceiveText" && !line.From.startsWith("$") && !line.Message.startsWith("$")) {
        return {
          ...line,
          From: "SOME SENDER",
          Message: "Some awesome message that is just to cool to show.",
        };
      }

      if (line.event === "WingInvite" || line.event === "WingAdd") {
        return {
          ...line,
          Name: "INVITATION_CMDR",
        };
      }

      if (line.event === "WingJoin") {
        return {
          ...line,
          Others: line.others.map(() => "OTHER CMDR"),
        }
      }

      if (line.event === "CrewLaunchFighter"
        || line.event === "CrewMemberJoins"
        || line.event === "CrewMemberQuits"
        || line.event === "CrewMemberRoleChange"
      ) {
        return {
          ...line,
          Name: "MULTICREW CREW"
        };
      }

      if (line.event === "JoinACrew" || line.event === "QuitACrewEvent") {
        return {
          ...line,
          Captain: "MULTICREW CAPTAIN",
        }
      }

      if (line.event === "KickCrewMemberEvent") {
        return {
          ...line,
          Crew: "MULTICREW CREW",
        }
      }

      return line;
    })
    .map(line => JSON.stringify(line))
    .join("\n");
}

</script>

<style lang="scss">

@font-face {
    font-family: "Outfit";
    font-weight: 400;
    src: url("./fonts/Outfit-Regular.ttf");
}

@font-face {
    font-family: "Outfit";
    font-weight: 600;
    src: url("./fonts/Outfit-Bold.ttf");
}

:global(html), :global(body), main {
    height: 100%;
    width: 100%;

    padding: 0;
    margin: 0;

    background-color: #000000;
    font-family: "Outfit", sans-serif;
}

main {
    display: grid;
    place-items: center;
}

section {
    max-width: 26rem;

    display: flex;
    flex-direction: column;
    gap: 2rem;
    padding: 2rem;

    background-color: #0f0f0f;
    border-radius: 0.5rem;
    color: white;
}

h1 {
    margin: 0;
    font-size: 3rem;
}

button {

}

</style>
