<script>
    export let handleFileUpload;
    export let handleClose;
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
    <h4>Upload CV</h4>
    <input type="file" on:change={handleFileUpload} />
    <button on:click={handleClose} class="btn btn-primary">Save</button>
    <button on:click={handleClose} class="btn btn-secondary">Close</button>
  </div>
  