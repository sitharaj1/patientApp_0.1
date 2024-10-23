<script lang="ts">
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  let firstName = '';
  let lastName = '';
  let gender = '';
  let birthDate = '';
  let phoneNumber = '';

  function handleSubmit() {
    const newPatient = {
      firstName,
      lastName,
      gender,
      birthDate,
      phoneNumber
    };

    dispatch('addPatient', newPatient);

    // Clear the form
    firstName = '';
    lastName = '';
    gender = '';
    birthDate = '';
    phoneNumber = '';
  }
</script>

<form on:submit|preventDefault={handleSubmit}>
  <div class="form-group">
    <label for="firstName">First Name</label>
    <input id="firstName" type="text" bind:value={firstName} required>
  </div>
  <div class="form-group">
    <label for="lastName">Last Name</label>
    <input id="lastName" type="text" bind:value={lastName} required>
  </div>
  <div class="form-group">
    <label for="gender">Gender</label>
    <select id="gender" bind:value={gender} required>
      <option value="">Select gender</option>
      <option value="male">Male</option>
      <option value="female">Female</option>
      <option value="other">Other</option>
    </select>
  </div>
  <div class="form-group">
    <label for="birthDate">Birth Date</label>
    <input id="birthDate" type="date" bind:value={birthDate} required>
  </div>
  <div class="form-group">
    <label for="phoneNumber">Phone Number</label>
    <input id="phoneNumber" type="tel" bind:value={phoneNumber} required>
  </div>
  <button type="submit" class="btn-primary">Add Patient</button>
</form>

<style>
  form {
    max-width: 400px;
    margin: 0 auto;
  }

  .form-group {
    margin-bottom: 1rem;
  }

  label {
    display: block;
    margin-bottom: 0.5rem;
  }

  input, select {
    width: 100%;
    padding: 0.5rem;
    font-size: 1rem;
    border: 1px solid #ccc;
    border-radius: 4px;
  }

  .btn-primary {
    background-color: #007bff;
    color: white;
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
  }

  .btn-primary:hover {
    background-color: #0056b3;
  }
</style>

