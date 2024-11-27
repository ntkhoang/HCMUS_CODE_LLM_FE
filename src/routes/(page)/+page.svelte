<script>
	let inputText = ''; // Bind this to the text box
  
	async function sendData() {
	  const url = 'https://2e9c-35-226-7-46.ngrok-free.app/ask';
		console.log('Sending data:', inputText); // Log the input text
	  try {
		const response = await fetch(url, {
		  method: 'POST',
		  headers: {
			'Content-Type': 'application/json',
		  },
		  body: JSON.stringify({ question: inputText }), // Match the API's expected structure
		});
  
		if (response.ok) {
		  const data = await response.json();
		  console.log('Response:', data.answer); // Log the API response
		} else {
		  console.error('Failed to send data:', response.statusText);
		}
	  } catch (error) {
		console.error('Error:', error);
	  }
	}
  </script>
  
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
  
  <div class="container">
	<input
	  type="text"
	  bind:value={inputText}
	  placeholder="Enter your question here"
	/>
	<button onclick={sendData}>Send to API</button>
  </div>