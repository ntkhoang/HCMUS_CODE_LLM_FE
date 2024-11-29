<script lang="ts">
  let inputText = $state("");
  let responsePromise: Promise<string> | null = $state(null);

  async function sendData() {
    const url = "https://2f4f-35-224-123-54.ngrok-free.app/ask";
    console.log("Sending data:", JSON.stringify({ question: inputText })); // Log the input text
    const response = await fetch(url, {
      method: "POST",
      headers: {
        "Content-Type": "application/json", // Set the Content-Type header
      },
      body: JSON.stringify({ question: inputText }), // Convert the body to JSON
    });

    if (response.ok) {
      const data = await response.json();
      console.log("Response:", data.answer);
      return data.answer;
    } else {
      throw new Error(response.statusText);
    }
  }
</script>

<div class="container">
  <input
    type="text"
    bind:value={inputText}
    placeholder="Enter your question here"
  />
  <button onclick={() => (responsePromise = sendData())}>Send to API</button>
</div>
{#await responsePromise}
  <p>Loading...</p>
{:then response}
  <p>{response}</p>
{:catch error}
  <p>Error: {error.message}</p>
{/await}

<style>
  .container {
    display: flex;
    flex-direction: column;
    gap: 10px;
    width: 300px;
    margin: auto;
    margin-top: 50px;
  }
</style>
