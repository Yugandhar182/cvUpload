<script>
	import { onMount } from "svelte";
	import { createEventDispatcher } from "svelte";
	import 'bootstrap/dist/css/bootstrap.min.css';
  
	let jsonData = [];
	let tableVisible = false;
	let selectedCandidate = null;
	let editedCandidate = null;
	let successMessage = "";
	let showSuccessMessage = false;
	let showDeleteConfirmation = false;
	let candidateToDelete = null;
	let showAddCandidatePopup = false;
	let newCandidate = { firstName: "", surname: "", email: "", mobile: "" };
	let uploadedFile = null;
  
	const dispatch = createEventDispatcher();
  
	onMount(async () => {
	  await fetchData();
	  tableVisible = true;
	});
  
	async function fetchData() {
	  const response = await fetch("https://api.recruitly.io/api/candidate/?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E");
	  const responseData = await response.json();
	  jsonData = responseData.data;
	}
  
	function handleTitleClick(candidate) {
	  selectedCandidate = candidate;
	  editedCandidate = { ...selectedCandidate };
	  dispatch("showPopup");
	}
  
	async function saveCandidate() {
	  // Update the selectedCandidate with the editedCandidate data
	  Object.assign(selectedCandidate, editedCandidate);
  
	  // Find the index of the selectedCandidate in the jsonData array
	  const index = jsonData.findIndex(candidate => candidate.id === selectedCandidate.id);
  
	  // Update the corresponding item in the jsonData array
	  jsonData[index] = { ...selectedCandidate };
  
	  // Send the updated candidate data to the API
	  const apiUrl = `https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`;
	  const response = await fetch(apiUrl, {
		method: 'POST',
		headers: {
		  'Content-Type': 'application/json'
		},
		body: JSON.stringify(selectedCandidate)
	  });
  
	  if (response.ok) {
		successMessage = "Your details have been successfully added.";
		showSuccessMessage = true;
	  } else {
		// Handle the case where the update request fails
		console.error("Failed to update candidate data.");
	  }
  
	  closePopup();
	}
  
	function closePopup() {
	  selectedCandidate = null;
	  editedCandidate = null;
	  newCandidate = { firstName: "", surname: "", email: "", mobile: "" };
	  uploadedFile = null;
	  showAddCandidatePopup = false;
	}
  
	function showDeleteWarning(candidate) {
	  candidateToDelete = candidate;
	  showDeleteConfirmation = true;
	}
  
	function cancelDelete() {
	  showDeleteConfirmation = false;
	  candidateToDelete = null;
	}
  
	async function deleteCandidate() {
	  // Remove the candidate from the jsonData array
	  jsonData = jsonData.filter(candidate => candidate.id !== candidateToDelete.id);
  
	  // Send the delete request to the API
	  const apiUrl = `https://api.recruitly.io/api/candidate/${candidateToDelete.id}?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`;
	  const response = await fetch(apiUrl, {
		method: 'DELETE'
	  });
  
	  if (response.ok) {
		successMessage = "Candidate deleted successfully.";
		showSuccessMessage = true;
	  } else {
		// Handle the case where the delete request fails
		console.error("Failed to delete candidate.");
	  }
  
	  cancelDelete();
	}
  
	function addCandidate() {
	  showAddCandidatePopup = true;
	}
  
	async function saveNewCandidate() {
	  // Send the new candidate data to the API
	  const apiUrl = `https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`;
	  const response = await fetch(apiUrl, {
		method: 'POST',
		headers: {
		  'Content-Type': 'application/json'
		},
		body: JSON.stringify(newCandidate)
	  });
  
	  if (response.ok) {
		successMessage = "New candidate added successfully.";
		showSuccessMessage = true;
		closePopup();
	  } else {
		// Handle the case where the create request fails
		console.error("Failed to create new candidate.");
	  }
	}
  
	function handleFileUpload(event) {
	  const file = event.target.files[0];
	  uploadedFile = file;
	}
  </script>
  
  <main class="container mt-4">
	<button class="btn btn-primary add-button" on:click={addCandidate}>Add Candidate</button>
  
	{#if tableVisible}
	  <div class="table-container">
		<table class="table">
		  <thead class="thead-light">
			<tr>
			  <th>First Name</th>
			  <th>Surname</th>
			  <th>Email</th>
			  <th>Mobile</th>
			  <th></th>
			</tr>
		  </thead>
		  <tbody>
			{#each jsonData as candidate}
			  <tr>
				<td>
				  <a href="#" on:click|preventDefault={() => handleTitleClick(candidate)}>{candidate.firstName}</a>
				  {#if showDeleteConfirmation && candidateToDelete === candidate}
					<div class="warning-popup">
					  <div class="warning-popup-content">
						<h1>Warning!</h1>
						<p>Are you sure you want to delete this candidate?</p>
						<button class="btn btn-danger" on:click|preventDefault={deleteCandidate}>Delete</button>
						<button class="btn btn-secondary" on:click|preventDefault={cancelDelete}>Cancel</button>
					  </div>
					</div>
				  {/if}
				</td>
				<td>{candidate.surname}</td>
				<td>{candidate.email}</td>
				<td>{candidate.mobile}</td>
				<td>
				  <button class="btn btn-danger btn-sm" on:click|preventDefault={() => showDeleteWarning(candidate)}>Delete</button>
				</td>
			  </tr>
			{/each}
		  </tbody>
		</table>
	  </div>
	{/if}
  </main>
  
  {#if selectedCandidate || showAddCandidatePopup}
  <div class="popup">
	<div class="popup-content">
	  <h1 style="color:blue;">Candidate Details</h1>
	  {#if showAddCandidatePopup}
	  <p>First Name: <input type="text" bind:value={newCandidate.firstName} placeholder="Enter first name" /></p>
	  <p>Surname: <input type="text" bind:value={newCandidate.surname} placeholder="Enter surname" /></p>
	  <p>Email: <input type="text" bind:value={newCandidate.email} placeholder="Enter email" /></p>
	  <p>Mobile: <input type="text" bind:value={newCandidate.mobile} placeholder="Enter mobile" /></p>
	  <p>CV Upload: <input type="file" on:change={handleFileUpload} accept=".pdf,.doc,.docx" /></p>
	  <div>
		<button class="btn btn-primary" on:click={saveNewCandidate}>Save</button>
		<button class="btn btn-secondary" on:click={closePopup}>Cancel</button>
	  </div>
	  {/if}
	  {#if selectedCandidate}
	  <p>First Name: <input type="text" bind:value={editedCandidate.firstName} placeholder="Enter first name" /></p>
	  <p>Surname: <input type="text" bind:value={editedCandidate.surname} placeholder="Enter surname" /></p>
	  <p>Email: <input type="text" bind:value={editedCandidate.email} placeholder="Enter email" /></p>
	  <p>Mobile: <input type="text" bind:value={editedCandidate.mobile} placeholder="Enter mobile" /></p>
	  <p>CV Upload: <input type="file" on:change={handleFileUpload} accept=".pdf,.doc,.docx" /></p>
	  <div>
		<button class="btn btn-primary" on:click={saveCandidate}>Save</button>
		<button class="btn btn-secondary" on:click={closePopup}>Cancel</button>
	  </div>
	  {/if}
	</div>
  </div>
  {/if}
  
  {#if showSuccessMessage}
  <div class="success-message">
	<p>{successMessage}</p>
	<button class="btn btn-secondary" on:click={() => showSuccessMessage = false}>Close</button>
  </div>
  {/if}
  
  <style>
	.popup {
	  position: fixed;
	  top: 0;
	  left: 0;
	  width: 100%;
	  height: 100%;
	  background-color: rgba(0, 0, 0, 0.5);
	  display: flex;
	  justify-content: center;
	  align-items: center;
	}
  
	.popup-content {
	  background-color: #ffffff;
	  padding: 40px;
	  border-radius: 10px;
	  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
	  max-width: 400px;
	  width: 90%;
	  text-align: center;
	}
  
	.popup-content h1 {
	  color: blue;
	  font-size: 24px;
	  margin-bottom: 25px;
	}
  
	.popup-content p {
	  margin-bottom: 10px;
	}
  
	.popup-content input {
	  width: 100%;
	  padding: 7px;
	  margin-bottom: 15px;
	}
  
	.popup-content button {
	  margin-top: 20px;
	}
  
	.success-popup {
	  position: fixed;
	  top: 0;
	  left: 0;
	  width: 100%;
	  height: 100%;
	  background-color: rgba(0, 0, 0, 0.5);
	  display: flex;
	  justify-content: center;
	  align-items: center;
	}
  
	.success-popup-content {
	  background-color: #ffffff;
	  padding: 40px;
	  border-radius: 10px;
	  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
	  max-width: 400px;
	  width: 90%;
	  text-align: center;
	}
  
	.success-popup-content p {
	  margin-bottom: 10px;
	}
  
	.success-popup-content button {
	  margin-top: 20px;
	}
	.success-popup-content h1 {
	  color: green;
	  font-size: 24px;
	  margin-bottom: 25px;
	}
	.success-message {
	  position: fixed;
	  top: 20px;
	  right: 20px;
	  padding: 10px;
	  background-color: #4caf50;
	  color: white;
	  border-radius: 5px;
	}
	.warning-popup {
	  position: absolute;
	  top: 0;
	  left: 0;
	  width: 100%;
	  height: 100%;
	  display: flex;
	  align-items: center;
	  justify-content: center;
	  background-color: rgba(0, 0, 0, 0.5);
	}
  
	.warning-popup-content {
	  max-width: 300px;
	  padding: 20px;
	  background-color: white;
	  border-radius: 5px;
	}
  
	.add-button {
	  margin-bottom: 20px;}
  </style>