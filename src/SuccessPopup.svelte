<script>
   
    import { createEventDispatcher } from "svelte";
  
    const dispatch = createEventDispatcher();
  </script>
  
  <style>
    @import 'bootstrap/dist/css/bootstrap.min.css';
  
    .popup {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background-color: rgb(34, 213, 18);
      padding: 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px rgba(31, 185, 36, 0.2);
    }
  </style>
  
  <div class="popup">
    <h4>CV Uploaded Successfully!</h4>
    <button on:click={() => dispatch("dismiss")} class="btn btn-primary">OK</button>
  </div>
  
  
  
  