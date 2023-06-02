<script>
	import { onMount } from "svelte";
	import { createEventDispatcher } from "svelte";
	import Table from "./Table.svelte";
	import Popup from "./Popup.svelte";
	import SuccessPopup from "./SuccessPopup.svelte";
	import DeletePopup from "./DeletePopup.svelte";
	import EditPopup from "./EditPopup.svelte";
  
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
	let isUploadSuccess = false;
	const dispatch = createEventDispatcher();
  
	onMount(async () => {
	  await fetchData();
	  tableVisible = true;
	});
  
	async function fetchData() {
	  const response = await fetch(
		"https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E"
	  );
	  const responseData = await response.json();
	  jsonData = responseData.data;
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
  
	  fetch(
		`https://api.recruitly.io/api/candidatecv/upload?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E&candidateId=${uploadJobId}`,
		{
		  method: "POST",
		  body: formData,
		}
	  )
		.then((response) => {
		  // Handle the response accordingly
		  if (response.ok) {
			console.log("CV uploaded successfully!");
			isUploadSuccess = true;
		  } else {
			console.error("CV upload failed.");
		  }
		})
		.catch((error) => {
		  console.error("CV upload error:", error);
		})
		.finally(() => {
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
	  isPopupVisible = false;
	  isDeletePopupVisible = false;
    isEditPopupVisible = false;
	 isUploadSuccess=false;
	}
  
	function handleDelete(jobId) {
	  deleteJobId = jobId;
	  isDeletePopupVisible = true;
	}
  
	function handleDeleteConfirm() {
	  fetch(
		`https://api.recruitly.io/api/candidate/${deleteJobId}?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E`,
		{
		  method: "DELETE",
		}
	  )
		.then((response) => {
		  // Handle the response accordingly
		  if (response.ok) {
			console.log("Job deleted successfully!");
			// Update the API URL to reflect the changes
			return fetchData();
		  } else {
			console.error("Job delete failed.");
		  }
		})
		.catch((error) => {
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
	  fetch("https://api.recruitly.io/api/candidate?apiKey=TEST1236C4CF23E6921C41429A6E1D546AC9535E", {
		method: "POST",
		body: JSON.stringify(editJob),
		headers: {
		  "Content-Type": "application/json",
		},
	  })
		.then((response) => {
		  // Handle the response accordingly
		  if (response.ok) {
			console.log("Job edited successfully!");
			// Update the API URL to reflect the changes
			return fetchData();
		  } else {
			console.error("Job edit failed.");
		  }
		})
		.catch((error) => {
		  console.error("Job edit error:", error);
		})
		.finally(() => {
		  isEditPopupVisible = false;
		});
	}
  </script>
  
  <main class="container mt-4">
	<Table {jsonData} {handleUploadCV} {handleDelete} {handleEdit} />
	{#if isPopupVisible && uploadJobId !== null}
	  <Popup {handleFileUpload} {handleClose} />
	{/if}
	{#if isUploadSuccess}
	  <SuccessPopup on:dismiss={handleClose} />
	{/if}
	{#if isDeletePopupVisible && deleteJobId !== null}
	  <DeletePopup
		{handleDeleteConfirm}
		{handleClose}
	  />
	{/if}
	{#if isEditPopupVisible && editJob !== null}
	  <EditPopup
		{editJob}
		{handleEditSave}
		{handleClose}
	  />
	{/if}
  </main>
  

  