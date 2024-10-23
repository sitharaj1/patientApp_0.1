<!-- This is the old version of the app -->
 <!--
<script lang="ts">
  let pageName = 'Search Patients';
  let pageDescription1 = '-> This page displays the details of the patient searched from Medblocks FHIR Server';
  let pageDescription2 = '-> Please input aleast one parameter';
  let pageDescription3 = '-> This page may also display multiple matching patient results';
  let pageDescriptionN = '';
  let showResults = false;
  let showResultsMatching = false;
  let InputPatientFirstName = '';
  let InputPatientLastName = '';
  let InputPatientUID = '';
  let InputPatientGender = 'Select gender';
  let errorMessage = '';

  // function handleInput(event: Event) {
  //   InputPatientFirstName = (event.currentTarget as HTMLInputElement).value;
  // }

//  const handleClick = (): void => {
//    pageDescriptionN = `Matching Patient details displayed here ${InputPatientFirstName} + ${InputPatientLastName} + ${InputPatientUID} + ${InputPatientGender}`;
 
//  };
// const patientDetails = async() => {
//   pageDescriptionN = `Matching Patient details displayed here ${InputPatientFirstName} + ${InputPatientLastName} + ${InputPatientUID} + ${InputPatientGender}`;
//   //const res = await fetch('https://fhir.medblocks.io/Patient');
//   const res = await fetch('https://fhir-bootcamp.medblocks.com/fhir/Patient');
//   //https://fhir-bootcamp.medblocks.com/fhir/Patient/7cd132ca-59e7-4ae8-afc1-286703d15047
//   const data = await res.json();
//   //console.log(data);
//   return data;
// };

  async function patientDetails() {
    pageDescriptionN = `Matching Patient details displayed here ${InputPatientFirstName} + ${InputPatientLastName} + ${InputPatientUID} + ${InputPatientGender}`;
    const res = await fetch('https://fhir-bootcamp.medblocks.com/fhir/Patient');
      // Remove the line trying to modify the style of the response
    //https://fhir-bootcamp.medblocks.com/fhir/Patient/7cd132ca-59e7-4ae8-afc1-286703d15047
    const dataFull = await res.json();
    // Process and display the data
    
    if (dataFull && dataFull.entry) {
      const patients = dataFull.entry.map((entry: { resource: any }) => entry.resource);
      //console.log('Patients:', patients);
      // Here you would typically update your UI with the patient data
      // For example, you might set a variable to hold the patient list:
      // patientList = patients;
      // For example, you might set a variable to hold the patient list:
      // patientList = patients;
      showResults = true;
    //console.log(data);
      return dataFull;
    } else {
      //console.error('No patient data found in the response');
      errorMessage = 'No patient data found in the response';
      showResultsMatching = false;
    }
};

async function patientDetailsMatching() {
    console.log("InputPatientGender - "+ InputPatientGender);
    if (!InputPatientFirstName && !InputPatientLastName && !InputPatientUID && !InputPatientGender) {
      alert("Please fill in at least one field before searching.");
      return;
    }

    pageDescriptionN = `Matching Patient details displayed here ${InputPatientFirstName} + ${InputPatientLastName} + ${InputPatientUID} + ${InputPatientGender}`;
    // console.log("InputPatientFirstName - "+ InputPatientFirstName);
    // console.log("InputPatientLastName - "+ InputPatientLastName);
    // console.log("InputPatientUID - "+ InputPatientUID);
    // console.log("InputPatientGender - "+ InputPatientGender);
    let resMatching;
    let queryString = '';
    if (InputPatientUID != '' && (InputPatientFirstName == '' && InputPatientLastName == '' && InputPatientGender == 'Select gender')) {
      queryString = 'https://fhir-bootcamp.medblocks.com/fhir/Patient/'+InputPatientUID;
      resMatching = await fetch(queryString);
    } else if (((InputPatientFirstName != '') || (InputPatientLastName != '')) && (InputPatientGender != 'Select gender')) {
      queryString = 'https://fhir-bootcamp.medblocks.com/fhir/Patient/?name:contains='+InputPatientFirstName+','+InputPatientLastName+'&gender='+InputPatientGender;
      resMatching = await fetch(queryString);
    } else if (InputPatientGender != 'Select gender') {
      queryString = 'https://fhir-bootcamp.medblocks.com/fhir/Patient/?gender='+InputPatientGender;
      resMatching = await fetch(queryString);
    }
    console.log("queryString - "+queryString);
    console.log("resMatching - "+resMatching);
    // } else {
    //   // If no specific criteria are met, fetch all patients
    //   res = await fetch('https://fhir-bootcamp.medblocks.com/fhir/Patient');
    // }
    let dataMatching;
    
  let errorMessage = '';
    if (resMatching) {
          dataMatching = await resMatching.json();
          // Process and display the data
          console.log("dataMatching - "+dataMatching);
          
          if (dataMatching && dataMatching.entry) {
            const patients = dataMatching.entry.map((entry: { resource: any }) => entry.resource);
            //console.log('Patients:', patients);
            // Here you would typically update your UI with the patient data
            // For example, you might set a variable to hold the patient list:
            // patientList = patients;
            // For example, you might set a variable to hold the patient list:
            // patientList = patients;
            showResultsMatching = true;
            return dataMatching;
      } else {
        //console.error('No patient data found in the response');
        errorMessage = 'No patient data found in the response';
        showResultsMatching = false;
      }
    }
};

</script>

<main>
	<h1>{pageName}</h1>
	<h4>{pageDescription1}</h4>
  <h4>{pageDescription2}</h4>
  <h4>{pageDescription3}</h4>
  <h4 color= #3366cc>{pageDescriptionN}</h4>
  <h4>-</h4>
  <h4>-</h4>
  <button type="submit"  class="btn-primary"  on:click={() => patientDetails()}>Get "All" Patients</button>;
  <h4>-</h4>
  <h4>-Fill in the following fields to search for a patient with a matching name or UID or gender</h4>
  <h4>-</h4>
  <table color= #808080>
    <tr>
      <h3>Patient First Name</h3>
      <td><input type="text" pattern="[a-zA-Z]+" maxlength="10" bind:value={InputPatientFirstName}/></td>
    </tr>
    <tr>
      <h3>Patient Last Name</h3>
      <td><input type="text" bind:value={InputPatientLastName} pattern="[a-zA-Z]+" maxlength="10"></td>
    </tr>
    <tr>
      <h3>Patient UID</h3>
      <td><input type="text" bind:value={InputPatientUID} pattern="[a-zA-Z0-9]+"></td>
      <h5> - *This criteria uniquely identifies a patient</h5>
    </tr>
    <tr>
      <label for="gender">Patient Gender</label>
      <td>
      <select id="gender" name="gender" bind:value={InputPatientGender}>
        <option value="">Select gender</option>
        <option value="male">Male</option>
        <option value="female">Female</option>
        <option value="non-binary">Non-binary</option>
        <option value="other">Other</option>
        <option value="prefer-not-to-say">Prefer not to say</option>
      </select>
      </td>
    </tr>
    <button class="btn-secondary" type="submit" on:click={() => patientDetailsMatching()}>Get "Matching" Patients</button>
  </table>
 

  {#if showResults}
    <div class="container" color= #808080>
      {#await patientDetails()}
        <p>Loading...</p>
      {:then data}
        {#each data.entry as { fullUrl, resource }}
          <div class="grid">
            <table>
                <tr>
                  <th>Patient Resource</th>
                  <th>Full URL</th>
                </tr>
            </table>
            <table>
                <tr>
                  <td>{resource.resourceType}</td>
                  <td>{fullUrl}</td>
                </tr>
            </table>
          </div>
        {/each}
      {:catch error}
        <p>Error: {error}</p>
      {/await}
    </div>
  {/if}

  {#if showResultsMatching}
  <div class="container" color= #808080>
    {#await patientDetailsMatching()}
      <p>Loading...</p>
    {:then dataMatching}
      {#each dataMatching.entry as { fullUrl, resource }}
        <div class="grid">
          <table>
              <tr>
                <th>Patient Resource</th>
                <th>Gender<th>
                <th>Full URL</th>
              </tr>
          </table>
          <table>
              <tr>
                <td>{resource.resourceType}</td>
                <td>{resource.gender}</td>
                <td>{fullUrl}</td>
              </tr>
          </table>
        </div>
      {/each}
    {:catch error}
      <p>Error: {error}</p>
    {/await}
  </div>
{/if}

{#if errorMessage}
<div class="error-message">
  {errorMessage}
</div>
{/if}

</main>

<style>
	main {
  	max-width: 1000px;
		margin: 100 auto;
    padding: 0px;
    text-align: justify;
    background-color: #ffffff;
    box-shadow: 100 100 1000px rgba(0, 0, 0, 0.1);
	}

	h1 {
		color: #ff3e00;
		text-transform: full-width;
		font-size: 4em;
		font-weight: 100;
    width: 1000px; /* or any other value, or use max-width for responsiveness */
    word-wrap: normal;
    overflow-wrap: anywhere;
    hyphens: auto;
	}
  
	h4 {
		color: #121111;
		text-transform: full-width;
		font-size: 1em;
		font-weight: 200;
    width: 1000px; /* or any other value, or use max-width for responsiveness */
    word-wrap: normal;
    overflow-wrap: anywhere;
    hyphens: auto;
	}

  .btn-primary {
    display: inline-block;
    padding: 10px 20px;
    background-color: #4CAF50; /* Green */
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
  }

  .btn-primary:hover {
    background-color: #45a049;
  }

  .btn-secondary {
    display: inline-block;
    padding: 10px 20px;
    background-color: #008CBA; /* Blue */
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 16px;
  }

  .btn-secondary:hover {
    background-color: #007399;
  }

  .grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }

  input[type="text"] {
    padding: 8px 8px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 16px;
    line-height: 1.5;
    width: 200px; 
  }

  table {
    border-collapse: collapse;
    background-color: #838fea;
    border: 1px solid #cccccc;
    width: 80%;
  }

  tr {
    border: 1px solid #ddd;
    padding: 1px;
    text-align: justify;
  }

  td {
    border: 1px solid #ddd;
    padding: 1px;
    text-align: justify;
  }

  .error-message {
    color: rgb(226, 161, 104);
    font-weight: bold;
    margin-top: 10px;
  }

</style>
-->