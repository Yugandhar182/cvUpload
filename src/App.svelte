<script>
	import { onMount } from "svelte";
	import { createEventDispatcher } from "svelte";
	import 'bootstrap/dist/css/bootstrap.min.css';
  
	let jsonData = [];
	let tableVisible = false;
	let selectedJob = null;
	let uploadJobId = null;
	let isPopupVisible = false;
	let file = null;
	let deleteJobId = null;
	let isDeletePopupVisible = false;
	let editJob = null;
	let isEditPopupVisible = false;
	let isUploadSuccess = false; // New state variable for success message
  
	const dispatch = createEventDispatcher();
  
	onMount(async () => {
	  await fetchData();
	  tableVisible = true;
	});
  
	async function fetchData() {
	  const response = await fetch("https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E");
	  const responseData = await response.json();
	  jsonData = responseData.data;
	}
  
	function handleTitleClick(job) {
	  selectedJob = job;
	  dispatch("showPopup");
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
			isUploadSuccess = true; // Set the flag to true
		  } else {
			console.error("CV upload failed.");
		  }
		})
		.catch(error => {
		  console.error("CV upload error:", error);
		})
		.finally(() => {
		  // Perform close logic
		  isPopupVisible = false;
		});
	}
  
	function handleSave() {
	  // Perform save logic
	  // In this case, we're updating the backend API URL with the file upload
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
			isUploadSuccess = true; // Set the flag to true
		  } else {
			console.error("CV upload failed.");
		  }
		})
		.catch(error => {
		  console.error("CV upload error:", error);
		})
		.finally(() => {
		  // Perform close logic
		  isPopupVisible = false;
		});
	}
  
	function handleClose() {
	  // Perform close logic
	  isPopupVisible = false;
	}
  
	function handleDelete(jobId) {
	  deleteJobId = jobId;
	  isDeletePopupVisible = true;
	}
  
	function handleDeleteConfirm() {
	  fetch(`https://api.recruitly.io/api/candidate/${deleteJobId}?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`, {
		method: "DELETE"
	  })
		.then(response => {
		  // Handle the response accordingly
		  if (response.ok) {
			console.log("Job deleted successfully!");
			// Update the API URL to reflect the changes
			return fetchData();
		  } else {
			console.error("Job delete failed.");
		  }
		})
		.catch(error => {
		  console.error("Job delete error:", error);
		})
		.finally(() => {
		  isDeletePopupVisible = false;
		});
	}
  
	function handleEdit(job) {
	  editJob = { ...job };
	  isEditPopupVisible = true;
	}
  
	function handleEditSave() {
	  // Perform save logic for the edited job
	  // Update the backend API URL with the updated job details
	  fetch(`https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`, {
		method: "POST",
		body: JSON.stringify(editJob),
		headers: {
		  "Content-Type": "application/json"
		}
	  })
		.then(response => {
		  // Handle the response accordingly
		  if (response.ok) {
			console.log("Job edited successfully!");
			// Update the API URL to reflect the changes
			return fetchData();
		  } else {
			console.error("Job edit failed.");
		  }
		})
		.catch(error => {
		  console.error("Job edit error:", error);
		})
		.finally(() => {
		  isEditPopupVisible = false;
		});
	}
  </script>
  
  <main class="container mt-4">
	{#if tableVisible}
	  <table class="table">
		<thead class="thead-light">
		  <tr>
			<th>ID</th>
			<th>firstName</th>
			<th>surname</th>
			<th>email</th>
			<th>mobile</th>
			<th>Actions</th>
		  </tr>
		</thead>
		<tbody>
		  {#each jsonData as job}
			<tr>
			  <td>{job.id}</td>
			  <td>{job.firstName}</td>
			  <td>{job.surname}</td>
			  <td>{job.email}</td>
			  <td>{job.mobile}</td>
			  <td>
				<button on:click={() => handleUploadCV(job.id)} class="btn btn-primary">Upload CV</button>
				<button on:click={() => handleEdit(job)} class="btn btn-info">Edit</button>
				<button on:click={() => handleDelete(job.id)} class="btn btn-danger">Delete</button>
			  </td>
			</tr>
		  {/each}
		</tbody>
	  </table>
	  {#if isPopupVisible && uploadJobId !== null}
		<div class="popup">
		  <h4>Upload CV</h4>
		  <input type="file" on:change={handleFileUpload} />
		  <button on:click={handleSave} class="btn btn-primary">Save</button>
		  <button on:click={handleClose} class="btn btn-secondary">Close</button>
		</div>
	  {/if}
	  {#if isUploadSuccess}
		<div class="popup success">
		  <h4>CV Uploaded Successfully!</h4>
		  <button on:click={() => isUploadSuccess = false} class="btn btn-primary">OK</button>
		</div>
	  {/if}
	  {#if isDeletePopupVisible && deleteJobId !== null}
		<div class="popup">
		  <h4>Confirm Delete</h4>
		  <p>Are you sure you want to delete this job?</p>
		  <button on:click={handleDeleteConfirm} class="btn btn-danger">Delete</button>
		  <button on:click={() => isDeletePopupVisible = false} class="btn btn-secondary">Cancel</button>
		</div>
	  {/if}
	  {#if isEditPopupVisible && editJob !== null}
		<div class="popup">
		  <h4>Edit Job</h4>
		  <label for="firstName">First Name:</label>
		  <input type="text" id="firstName" bind:value={editJob.firstName} class="form-control" />
		  <label for="surname">Surname:</label>
		  <input type="text" id="surname" bind:value={editJob.surname} class="form-control" />
		  <label for="email">Email:</label>
		  <input type="email" id="email" bind:value={editJob.email} class="form-control" />
		  <label for="mobile">Mobile:</label>
		  <input type="text" id="mobile" bind:value={editJob.mobile} class="form-control" />
		  <button on:click={handleEditSave} class="btn btn-primary">Save</button>
		  <button on:click={() => isEditPopupVisible = false} class="btn btn-secondary">Cancel</button>
		</div>
	  {/if}
	{/if}
  </main>
  
  <style>
	.popup {
	  position: fixed;
	  top: 50%;
	  left: 50%;
	  transform: translate(-50%, -50%);
	  background-color: white;
	  padding: 20px;
	  border-radius: 8px;
	  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
	}
  
	.success {
	  background-color: lightgreen;
	}
  </style>
  