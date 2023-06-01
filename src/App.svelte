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
			  </td>
			</tr>
		  {/each}
		</tbody>
	  </table>
	  {#if isPopupVisible && uploadJobId !== null}
		<div class="popup">
		  <h4>Upload CV for ID {uploadJobId}</h4>
		  <input type="file" on:change={handleFileUpload} />
		  <button on:click={handleSave} class="btn btn-primary">Save</button>
		  <button on:click={handleClose} class="btn btn-secondary">Close</button>
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
	  border: 1px solid black;
	  z-index: 9999;
	}
  </style>
  