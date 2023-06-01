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
	let newCandidate = { id: "", firstName: "", surname: "", email: "", mobile: "" };
	let showCVUploadPopup = false;
	let file = null;
	
	l
  
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
	  newCandidate = { id: "", firstName: "", surname: "", email: "", mobile: "" };
  
	  showAddCandidatePopup = false;
	  showCVUploadPopup = false;
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
  
	function handleAddCandidateClick() {
	  showAddCandidatePopup = true;
	}
  
	async function addCandidate() {
	  // Generate a new candidate ID
	  const newId = Date.now().toString();
  
	  // Assign the new ID to the newCandidate object
	  newCandidate.id = newId;
  
	  // Add the newCandidate to the jsonData array
	  jsonData.push(newCandidate);
  
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
	  } else {
		// Handle the case where the add request fails
		console.error("Failed to add candidate.");
	  }
  
	  closePopup();
	}
  function handleUploadCV(jobId) {
	  uploadJobId = jobId;
	  isPopupVisible = true;
	}
  
	function handleFileUpload(event) {
	  file = event.target.files[0];
	  // Perform further actions with the uploaded file
  
	  // Example: Update the backend API URL with the file upload
	  const formData = new FormData();
	  formData.append("file", file);
  
	  fetch(`https://api.recruitly.io/api/candidatecv/upload?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E&candidateId=${uploadJobId}`, {
		method: "POST",
		body: formData
	  })
		.then(response => {
		  // Handle the response accordingly
		  if (response.ok) {
			console.log("CV uploaded successfully!");
		  } else {
			console.error("CV upload failed.");
		  }
		})
		.catch(error => {
		  console.error("CV upload error:", error);
		});
	}
  
	function handleSave() {
	  // Perform save logic
	  // In this case, we're updating the backend API URL in the handleFileUpload function
	  isPopupVisible = false;
	}
  
	function handleClose() {
	  // Perform close logic
	  isPopupVisible = false;
	}
  </script>
  
  {#if tableVisible}
	<table class="table">
	  <thead>
		<tr>
		  <th scope="col">ID</th>
		  <th scope="col">First Name</th>
		  <th scope="col">Surname</th>
		  <th scope="col">Email</th>
		  <th scope="col">Mobile</th>
		  <th scope="col">Actions</th>
		</tr>
	  </thead>
	  <tbody>
		{#each jsonData as candidate}
		  <tr>
			<td>{candidate.id}</td>
			<td>{candidate.firstName}</td>
			<td>{candidate.surname}</td>
			<td>{candidate.email}</td>
			<td>{candidate.mobile}</td>
			<td>
			  <button class="btn btn-primary btn-sm" on:click|preventDefault={() => handleTitleClick(candidate)}>Edit</button>
			  <button class="btn btn-danger btn-sm" on:click|preventDefault={() => showDeleteWarning(candidate)}>Delete</button>
			  <button class="btn btn-primary btn-sm" on:click|preventDefault={() => handleCVUploadClick(candidate.id)}>CV Upload</button>
			</td>
		  </tr>
		{/each}
	  </tbody>
	</table>
	<br />
  
	<button class="btn btn-primary btn-sm" on:click={handleAddCandidateClick}>Add Candidate</button>
  {/if}
  
  {#if showAddCandidatePopup}
	<div class="popup">
	  <div class="popup-inner">
		<h2>Add Candidate</h2>
		<label for="firstName">First Name:</label>
		<input type="text" id="firstName" bind:value={newCandidate.firstName} />
		<label for="surname">Surname:</label>
		<input type="text" id="surname" bind:value={newCandidate.surname} />
		<label for="email">Email:</label>
		<input type="text" id="email" bind:value={newCandidate.email} />
		<label for="mobile">Mobile:</label>
		<input type="text" id="mobile" bind:value={newCandidate.mobile} />
		<button class="btn btn-primary" on:click={addCandidate}>Save</button>
		<button class="btn btn-secondary" on:click={closePopup}>Cancel</button>
	  </div>
	</div>
  {/if}
  
  {#if selectedCandidate}
	<div class="popup">
	  <div class="popup-inner">
		<h2>Edit Candidate</h2>
		<label for="firstName">First Name:</label>
		<input type="text" id="firstName" bind:value={editedCandidate.firstName} />
		<label for="surname">Surname:</label>
		<input type="text" id="surname" bind:value={editedCandidate.surname} />
		<label for="email">Email:</label>
		<input type="text" id="email" bind:value={editedCandidate.email} />
		<label for="mobile">Mobile:</label>
		<input type="text" id="mobile" bind:value={editedCandidate.mobile} />
		<button class="btn btn-primary" on:click={saveCandidate}>Save</button>
		<button class="btn btn-secondary" on:click={closePopup}>Cancel</button>
	  </div>
	</div>
  {/if}
  
  {#if showCVUploadPopup}
	<div class="popup">
	  <div class="popup-inner">
		<h2>Upload CV</h2>
		<input type="file" on:change={event => { file = event.target.files[0]; fileName = file ? file.name : ''; }} />
		<p>{fileName}</p>
		<button class="btn btn-primary" on:click={saveCV}>Upload</button>
		<button class="btn btn-secondary" on:click={closePopup}>Cancel</button>
	  </div>
	</div>
  {/if}
  
  {#if showDeleteConfirmation}
	<div class="popup">
	  <div class="popup-inner">
		<h2>Confirm Delete</h2>
		<p>Are you sure you want to delete this candidate?</p>
		<button class="btn btn-danger" on:click={deleteCandidate}>Delete</button>
		<button class="btn btn-secondary" on:click={cancelDelete}>Cancel</button>
	  </div>
	</div>
  {/if}
  
  {#if showSuccessMessage}
	<div class="success-message">
	  <p>{successMessage}</p>
	  <button class="btn btn-primary btn-sm" on:click={() => showSuccessMessage = false}>Close</button>
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
	  align-items: center;
	  justify-content: center;
	}
  
	.popup-inner {
	  background-color: white;
	  padding: 20px;
	  border-radius: 5px;
	  max-width: 400px;
	  text-align: center;
	}
  
	.success-message {
	  position: fixed;
	  top: 20px;
	  right: 20px;
	  background-color: green;
	  padding: 10px;
	  color: white;
	  border-radius: 5px;
	  display: flex;
	  align-items: center;
	}
  </style>
  