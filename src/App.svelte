<main>
  <section>
    <h1>Anonymize journals</h1>

    {#if !working}
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
          Name: "TEST",
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
    background-color: #0f0f0f;

    display: flex;
    flex-direction: column;
    gap: 2rem;
    padding: 2rem;

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
