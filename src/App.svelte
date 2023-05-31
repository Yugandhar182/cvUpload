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
  let showCVUploadPopup = false; // Added property for the CV Upload popup

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

    showAddCandidatePopup = false;
    showCVUploadPopup = false; // Close the CV Upload popup
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

    showDeleteConfirmation = false;
    candidateToDelete = null;
  }

  function addCandidate() {
    newCandidate = { firstName: "", surname: "", email: "", mobile: "" };
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

      // Fetch the updated candidate list
      await fetchData();
    } else {
      // Handle the case where the create request fails
      console.error("Failed to create new candidate.");
    }

    closePopup();
  }
</script>

<style>
  .popup {
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 9999;
  }

  .popup-content {
    background-color: #fff;
    padding: 20px;
    border-radius: 4px;
    max-width: 500px;
    width: 100%;
  }

  .popup-content h1 {
    color: blue;
    font-size: 24px;
    margin-bottom: 10px;
  }

  .popup-content p {
    margin-bottom: 10px;
  }

  .popup-content input {
    margin-bottom: 10px;
    padding: 5px;
    width: 100%;
  }

  .popup-content button {
    padding: 5px 10px;
  }
</style>

{#if tableVisible}
  <button class="btn btn-primary" on:click|preventDefault={addCandidate}>Add Candidate</button>
  <table class="table table-bordered">
    <thead>
      <tr>
        <th>First Name</th>
        <th>Surname</th>
        <th>Email</th>
        <th>Mobile</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      {#each jsonData as candidate}
        <tr>
          <td>
            <a href="#" on:click|preventDefault={() => handleTitleClick(candidate)}>{candidate.firstName}</a>
            <button class="btn btn-primary btn-sm" on:click|preventDefault={() => showCVUploadPopup = true}>CV Upload</button>
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
{/if}

{#if selectedCandidate}
  <div class="popup">
    <div class="popup-content">
      <h1>Edit Candidate</h1>
      <p>First Name:</p>
      <input type="text" bind:value={editedCandidate.firstName}>

      <p>Surname:</p>
      <input type="text" bind:value={editedCandidate.surname}>

      <p>Email:</p>
      <input type="text" bind:value={editedCandidate.email}>

      <p>Mobile:</p>
      <input type="text" bind:value={editedCandidate.mobile}>

      <button class="btn btn-primary" on:click|preventDefault={saveCandidate}>Save</button>
      <button class="btn btn-secondary" on:click|preventDefault={closePopup}>Cancel</button>
    </div>
  </div>
{/if}

{#if showAddCandidatePopup}
  <div class="popup">
    <div class="popup-content">
      <h1>Add Candidate</h1>
      <p>First Name:</p>
      <input type="text" bind:value={newCandidate.firstName}>

      <p>Surname:</p>
      <input type="text" bind:value={newCandidate.surname}>

      <p>Email:</p>
      <input type="text" bind:value={newCandidate.email}>

      <p>Mobile:</p>
      <input type="text" bind:value={newCandidate.mobile}>

      <button class="btn btn-primary" on:click|preventDefault={saveNewCandidate}>Save</button>
      <button class="btn btn-secondary" on:click|preventDefault={closePopup}>Cancel</button>
    </div>
  </div>
{/if}

{#if showDeleteConfirmation}
  <div class="popup">
    <div class="popup-content">
      <h1>Confirm Delete</h1>
      <p>Are you sure you want to delete this candidate?</p>
      <button class="btn btn-danger" on:click|preventDefault={deleteCandidate}>Delete</button>
      <button class="btn btn-secondary" on:click|preventDefault={cancelDelete}>Cancel</button>
    </div>
  </div>
{/if}

{#if showSuccessMessage}
  <div class="popup">
    <div class="popup-content">
      <h1>Success!</h1>
      <p>{successMessage}</p>
      <button class="btn btn-primary" on:click|preventDefault={() => showSuccessMessage = false}>OK</button>
    </div>
  </div>
{/if}

{#if showCVUploadPopup}
  <div class="popup">
    <div class="popup-content">
      <h1>CV Upload</h1>
      <p>Choose File:</p>
      <input type="file" accept=".pdf, .doc, .docx">

      <button class="btn btn-primary" on:click|preventDefault={saveCV}>Save</button>
      <button class="btn btn-secondary" on:click|preventDefault={closePopup}>Cancel</button>
    </div>
  </div>
{/if}

