<script>
    export let handleEditSave;
    export let handleClose;
    export let editJob;
  </script>
  
  <style>
    @import 'bootstrap/dist/css/bootstrap.min.css';
  
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
  </style>
  
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
    <button on:click={handleClose} class="btn btn-secondary">Cancel</button>
  </div>
  