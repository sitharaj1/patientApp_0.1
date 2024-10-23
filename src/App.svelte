<script lang="ts">
	import { onMount } from 'svelte';
	import AddPatientForm from './AddPatientForm.svelte';
	import { slide } from 'svelte/transition';
	//;

	let pageName = 'Search Patients';
	let pageDescription = "This page displays patient details from Medblocks FHIR Server. ";
	let getAllPatientsDescription = "Click here to see all patients on the Server";
	let showResults = false;
	let patientData = { firstName: "", lastName: "", uid: "", gender: "" };
	let errorMessage = "";
	let patients: string | any[] = [];
	let patientCount = 0;

	const BASE_URL = "https://fhir-bootcamp.medblocks.com/fhir/Patient";

	let newPatient = { firstName: "", lastName: "", gender: "", birthDate: "", phoneNumber: "" };
	let showAddPatientForm = false;

	let isNavOpen = false;
	let activeSection = 'search'; // Default to search view

	let expandedPatientId: string | null = null;

	function toggleNav() {
		isNavOpen = !isNavOpen;
	}

	function setActiveSection(section: string) {
		activeSection = section;
		isNavOpen = false; // Close nav after selection on mobile
	}

	function togglePatientDetails(patientId: string) {
		expandedPatientId = expandedPatientId === patientId ? null : patientId;
	}

	//This piece of code is for fetching all patients on mount. 
	// onMount(() => {
	// 	fetchAllPatients();
	// });

	async function fetchAllPatients() {
		try {
			const response = await fetch(BASE_URL);
			const data = await response.json();
			patients = data.entry?.map((entry: { resource: any; }) => entry.resource) || [];
			patientCount = patients.length;
			showResults = true; // Show results after fetching
		} catch (error) {
			errorMessage = "Failed to fetch patients";
			showResults = false; // Hide results if there's an error
			patientCount = 0;
		}
	}

	async function searchPatients() {
		const validationError = validateForm();
		if (validationError) {
			alert(validationError);
			return;
		}

		if (Object.values(patientData).every((val) => !val)) {
			alert("Please fill in at least one field before searching.");
			return;
		}

		let queryString = BASE_URL;

		if (patientData.uid) {
			queryString += `/${patientData.uid}`;
		} else {
			const params = [];
			if (patientData.firstName || patientData.lastName) {
				params.push(
					`name:contains=${patientData.firstName},${patientData.lastName}`
					);
			}
			if (patientData.gender) {
				params.push(`gender=${patientData.gender}`);
			}
			if (params.length) {
				queryString += `?${params.join("&")}`;
			}
		}

		try {
			const response = await fetch(queryString);
			const data = await response.json();
			patients = data.entry?.map((entry: { resource: any; }) => entry.resource) || [data];
			patientCount = patients.length;
			showResults = true; // Show results after searching
			if (response.status == 410 || response.status == 404) {
				alert(`Patient unknown, please check the ID and try again.`);
				showResults = false; // Hide results if there's an error
				patientCount = 0;
			}
			errorMessage = "";
		} catch (error) {
			errorMessage = "Failed to fetch matching patients";
			showResults = false; // Hide results if there's an error
			patientCount = 0;
		}
	}
//::
	async function deletePatient(patientId: string) {
		if (!confirm(`Are you sure you want to delete patient with ID ${patientId}?`)) {
			return;
		}

		try {
			const response = await fetch(`${BASE_URL}/${patientId}`, {
				method: 'DELETE',
				headers: {
					'Content-Type': 'application/json',
					// Add any necessary authentication headers here
				},
			});

			if (!response.ok) {
				console.error(`HTTP error! status: `);
				throw new Error(`HTTP error! status: ${response.status}`);
			}

			// Remove the deleted patient from the patients array
			patients = patients.filter(patient => patient.id !== patientId);
			
			alert(`Patient with ID ${patientId} has been successfully deleted.`);
		} catch (error) {
			console.error('Error deleting patient:', error);
			alert(`Failed to delete patient with ID ${patientId}. Please try again.`);
		}
	}

	// Validation functions
	function isValidName(name: string): boolean {
		return /^[a-zA-Z\s-]{1,50}$/.test(name);
	}

	function isValidUUID(uuid: string): boolean {
		const uuidRegex = /^[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[1-5][0-9a-fA-F]{3}-[89abAB][0-9a-fA-F]{3}-[0-9a-fA-F]{12}$/;
		return uuidRegex.test(uuid);
	}

	function validateForm(): string {
		if (patientData.firstName && !isValidName(patientData.firstName)) {
			return "Invalid first name. Please use only letters, spaces, and hyphens (max 50 characters).";
		}
		if (patientData.lastName && !isValidName(patientData.lastName)) {
			return "Invalid last name. Please use only letters, spaces, and hyphens (max 50 characters).";
		}
		if (patientData.uid && !isValidUUID(patientData.uid)) {
			return "Invalid UID. Please enter a valid UUID.";
		}
		return "";
	}

	async function addPatient() {
		const validationError = validateNewPatientForm();
		if (validationError) {
			alert(validationError);
			return;
		}

		const patientResource = {
			resourceType: "Patient",
			name: [
				{
					use: "official",
					family: newPatient.lastName,
					given: [newPatient.firstName]
				}
			],
			gender: newPatient.gender,
			birthDate: newPatient.birthDate,
			telecom: [
				{
					system: "phone",
					value: newPatient.phoneNumber,
					use: "mobile"
				}
			]
		};

		try {
			const response = await fetch(BASE_URL, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
					// Add any necessary authentication headers here
				},
				body: JSON.stringify(patientResource)
			});

			if (!response.ok) {
				throw new Error(`HTTP error! status: ${response.status}`);
			}

			const result = await response.json();
			alert(`Patient added successfully with ID: ${result.id}`);
			newPatient = { firstName: "", lastName: "", gender: "", birthDate: "", phoneNumber: "" };
			showAddPatientForm = false;
		} catch (error) {
			console.error('Error adding patient:', error);
			alert('Failed to add patient. Please try again.');
		}
	}

	function validateNewPatientForm(): string {
		if (!newPatient.firstName || !newPatient.lastName) {
			return "First name and last name are required.";
		}
		if (!isValidName(newPatient.firstName) || !isValidName(newPatient.lastName)) {
			return "Invalid name. Please use only letters, spaces, and hyphens (max 50 characters).";
		}
		if (!newPatient.gender) {
			return "Please select a gender.";
		}
		if (!newPatient.birthDate) {
			return "Birth date is required.";
		}
		if (!newPatient.phoneNumber) {
			return "Phone number is required.";
		}
		if (!/^\d{5,15}$/.test(newPatient.phoneNumber)) {
			return "Invalid phone number. Please enter 5-15 digits.";
		}
		return "";
	}
</script>

<main class:nav-open={isNavOpen}>
	<nav class:open={isNavOpen}>
		<button class="nav-toggle" on:click={toggleNav}>
			{isNavOpen ? '✕' : '☰'}
		</button>
		<ul>
			<li><a href="#home" on:click|preventDefault={() => setActiveSection('home')}>Home</a></li>
			<li><a href="#search" class:active={activeSection === 'search'} on:click|preventDefault={() => setActiveSection('search')}>Search Patients</a></li>
			<li><a href="#add" class:active={activeSection === 'add'} on:click={() => showAddPatientForm = !showAddPatientForm}>Add Patient</a></li>
			<li><a href="#reports" on:click|preventDefault={() => setActiveSection('reports')}>Reports</a></li>
			<li><a href="#settings" on:click|preventDefault={() => setActiveSection('settings')}>Settings</a></li>
		</ul>
	</nav>

	<div class="content">
		<h1>{pageName}</h1>
		<p>{pageDescription}</p>
		<table class="main-table">
			<tr>
				<!--<td>	
					<button class="btn-primary" on:click={() => showAddPatientForm = !showAddPatientForm}>
					{showAddPatientForm ? 'Hide Add Patient Form' : 'Show Add Patient Form'}
				</button></td>
				-->
				<td class="button-cell">
					<p>{getAllPatientsDescription}</p>
					<p>-</p>
					<p>-</p>
					<div class="button-container">
						<button class="btn-primary" on:click={fetchAllPatients}>Get All Patients</button>
					</div>
				</td>
				<td>
					<h2>Search for Patients using specific search parameters</h2>
					<h2>Please input at least one parameter.</h2>
					<form on:submit|preventDefault={searchPatients}>
						<input
							type="text"
							placeholder="First Name"
							bind:value={patientData.firstName}
							class:invalid={patientData.firstName && !isValidName(patientData.firstName)}
						/>
						<input
							type="text"
							placeholder="Last Name"
							bind:value={patientData.lastName}
							class:invalid={patientData.lastName && !isValidName(patientData.lastName)}
						/>
						<input
							type="text"
							placeholder="UID"
							bind:value={patientData.uid}
							class:invalid={patientData.uid && !isValidUUID(patientData.uid)}
						/>
						<select bind:value={patientData.gender}>
							<option value="">Select gender</option>
							<option value="male">Male</option>
							<option value="female">Female</option>
							<option value="other">Other</option>
						</select>
						<div class="button-container">
							<button type="submit" class="btn-secondary">Search Patients</button>
						</div>
					</form>
				</td>

			</tr>
		</table>
		<table class="main-table">
			{#if showResults}
			<tr>
			<div class="container">
				<p class="patient-count">Total patients found: {patientCount}</p>
				{#if patients.length}
					<div class="table-responsive">
						<table class="collapsible-table">
							<thead>
								<tr>
									<th>Name</th>
									<th>Gender</th>
									<th>ID</th>
									<th>Actions</th>
								</tr>
							</thead>
							<tbody>
								{#each patients as patient}
									<tr class="patient-row" class:expanded={expandedPatientId === patient.id}>
										<td>{patient.name?.[0]?.given?.join(" ")} {patient.name?.[0]?.family}</td>
										<td>{patient.gender}</td>
										<td>{patient.id}</td>
										<td>
											<button class="btn-expand" on:click={() => togglePatientDetails(patient.id)}>
												{expandedPatientId === patient.id ? '▲' : '▼'}
											</button>
											<button class="btn-delete" on:click={() => deletePatient(patient.id)}>Delete</button>
										</td>
									</tr>
									{#if expandedPatientId === patient.id}
										<tr class="patient-details" transition:slide|local>
											<td colspan="4">
												<div class="details-grid">
													<div><strong>Birth Date:</strong> {patient.birthDate || 'N/A'}</div>
													<div><strong>Phone:</strong> {patient.telecom?.[0]?.value || 'N/A'}</div>
													<div><strong>Address:</strong> {patient.address?.[0]?.line?.join(', ') || 'N/A'}</div>
													<div><strong>City:</strong> {patient.address?.[0]?.city || 'N/A'}</div>
													<div><strong>State:</strong> {patient.address?.[0]?.state || 'N/A'}</div>
													<div><strong>Country:</strong> {patient.address?.[0]?.country || 'N/A'}</div>
												</div>
											</td>
										</tr>
									{/if}
								{/each}
							</tbody>
						</table>
					</div>
				{:else}
					<p>No patients found.</p>
				{/if}
			</div>
			{/if}

			{#if errorMessage}
				<div class="error-message">{errorMessage}</div>
			{/if}
		</table>
	</div>
	<div class="content" transition:slide|local>
		<table class="main-table">
			{#if showAddPatientForm}
				<div class="add-patient-form">
					<h2>Add New Patient</h2>
					<!-- svelte-ignore missing-declaration -->
					<form on:submit|preventDefault={addPatient}>
						<input
							type="text"
							placeholder="First Name"
							bind:value={newPatient.firstName}
							required
						/>
						<input
							type="text"
							placeholder="Last Name"
							bind:value={newPatient.lastName}
							required
						/>
						<select bind:value={newPatient.gender} required>
							<option value="">Select gender</option>
							<option value="male">Male</option>
							<option value="female">Female</option>
							<option value="other">Other</option>
						</select>
						<input
							type="date"
							bind:value={newPatient.birthDate}
							required
						/>
						<input
							type="tel"
							placeholder="Phone Number"
							bind:value={newPatient.phoneNumber}
							required
						/>
						<button type="submit" class="btn-secondary">Add Patient</button>
					</form>
				</div>
			{/if}
		</table>
	</div>
</main>

<style>
	main {
		display: flex;
		max-width: 100%;
		margin: 0;
		padding: 0;
		transition: margin-left 0.3s ease-in-out;
	}

	main.nav-open {
		margin-left: 250px;
	}

	nav {
		position: fixed;
		left: -250px;
		top: 0;
		bottom: 0;
		width: 250px;
		background-color: #f8f9fa;
		transition: left 0.3s ease-in-out;
		box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
		z-index: 1000;
	}

	nav.open {
		left: 0;
	}

	.nav-toggle {
		position: absolute;
		top: 10px;
		right: -40px;
		background: #f8f9fa;
		border: none;
		font-size: 24px;
		cursor: pointer;
		padding: 5px 10px;
		border-radius: 0 5px 5px 0;
	}

	nav ul {
		list-style-type: none;
		padding: 0;
		margin-top: 50px;
	}

	nav ul li a {
		display: block;
		padding: 15px 20px;
		color: #333;
		text-decoration: none;
		transition: background-color 0.3s;
	}

	nav ul li a:hover, nav ul li a.active {
		background-color: #e9ecef;
	}

	.content {
		flex-grow: 1;
		padding: 20px;
		transition: margin-left 0.3s ease-in-out;
	}

	h1 {
		color: #ff3e00;
	}

	.main-table {
		width: 100%;
		border-collapse: separate;
		border-spacing: 0;
		border: 2px solid #ddd;
	}

	.main-table td {
		vertical-align: top;
		padding: 20px;
	}

	.button-cell {
		text-align: center;
	}

	.button-container {
		display: flex;
		justify-content: center;
		margin-top: 10px;
	}

	.btn-primary,
	.btn-secondary {
		padding: 10px 20px;
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		font-size: 16px;
	}

	.btn-primary {
		background-color: #4caf50;
	}

	.btn-secondary {
		background-color: #008cba;
	}

	table {
		width: 100%;
		border-collapse: collapse;
	}

	th,
	td {
		border: 1px solid #ddd;
		padding: 8px;
		text-align: left;
	}

	th {
		background-color: #f2f2f2;
	}

	.error-message {
		color: red;
		font-weight: bold;
	}

	input,
	select {
		padding: 8px 12px;
		font-size: 14px;
		width: 200px;
		max-width: 100%;
		border: 1px solid #ccc;
		border-radius: 4px;
		transition: border-color 0.3s ease;
	}

	input:focus,
	select:focus {
		outline: none;
		border-color: #4caf50;
		box-shadow: 0 0 5px rgba(76, 175, 80, 0.5);
	}

	input:hover,
	select:hover {
		border-color: #999;
	}

	/* Style for the placeholder text */
	input::placeholder {
		color: #999;
	}

	/* Style for the select dropdown arrow */
	select {
		appearance: none;
		background-image: url("data:image/svg+xml;charset=US-ASCII,%3Csvg%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20width%3D%22292.4%22%20height%3D%22292.4%22%3E%3Cpath%20fill%3D%22%23007CB2%22%20d%3D%22M287%2069.4a17.6%2017.6%200%200%200-13-5.4H18.4c-5%200-9.3%201.8-12.9%205.4A17.6%2017.6%200%200%200%200%2082.2c0%205%201.8%209.3%205.4%2012.9l128%20127.9c3.6%203.6%207.8%205.4%2012.8%205.4s9.2-1.8%2012.8-5.4L287%2095c3.5-3.5%205.4-7.8%205.4-12.8%200-5-1.9-9.2-5.5-12.8z%22%2F%3E%3C%2Fsvg%3E");
		background-repeat: no-repeat;
		background-position: right 12px top 50%;
		background-size: 12px auto;
		padding-right: 30px;
	}

	input.invalid {
		border-color: #dc3545;
	}

	input.invalid:focus {
		box-shadow: 0 0 0 0.2rem rgba(220, 53, 69, 0.25);
	}

	.patient-count {
		font-weight: bold;
		text-align: center;
		margin-bottom: 15px;
		font-size: 18px;
		color: #4a4a4a;
	}

	.add-patient-form {
		margin-top: 20px;
		padding: 20px;
		border: 1px solid #ddd;
		border-radius: 4px;
	}

	.add-patient-form form {
		display: flex;
		flex-direction: column;
		gap: 10px;
	}

	.add-patient-form input,
	.add-patient-form select {
		padding: 8px;
		font-size: 14px;
	}

	nav ul li a.active {
		background-color: #e9ecef;
		font-weight: bold;
	}

	.patient-list {
		display: flex;
		flex-direction: column;
		gap: 10px;
	}

	.patient-card {
		border: 1px solid #dee2e6;
		border-radius: 4px;
		overflow: hidden;
	}

	.patient-summary {
		display: flex;
		justify-content: space-between;
		align-items: center;
		padding: 10px;
		background-color: #f8f9fa;
		cursor: pointer;
	}

	.patient-summary:hover {
		background-color: #e9ecef;
	}

	.patient-details {
		padding: 15px;
		background-color: #ffffff;
		border-top: 1px solid #dee2e6;
	}

	.details-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
		gap: 10px;
		margin-bottom: 15px;
	}

	.btn-expand {
		background: none;
		border: none;
		font-size: 1.2em;
		cursor: pointer;
		padding: 0 10px;
	}

	.btn-delete {
		padding: 5px 10px;
		background-color: #dc3545;
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		transition: background-color 0.15s ease-in-out;
	}

	.btn-delete:hover {
		background-color: #c82333;
	}

	@media screen and (max-width: 600px) {
		.patient-summary {
			flex-direction: column;
			align-items: flex-start;
		}

		.patient-summary span {
			margin-bottom: 5px;
		}

		.details-grid {
			grid-template-columns: 1fr;
		}
	}

	.collapsible-table {
		width: 100%;
		border-collapse: separate;
		border-spacing: 0;
	}

	.collapsible-table th,
	.collapsible-table td {
		padding: 10px;
		border-bottom: 1px solid #dee2e6;
	}

	.collapsible-table th {
		background-color: #f8f9fa;
		font-weight: bold;
	}

	.patient-row {
		cursor: pointer;
	}

	.patient-row:hover {
		background-color: #f8f9fa;
	}

	.patient-row.expanded {
		background-color: #e9ecef;
	}

	.patient-details td {
		padding: 0;
	}

	.details-grid {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
		gap: 10px;
		padding: 15px;
		background-color: #ffffff;
	}

	.btn-expand,
	.btn-delete {
		padding: 5px 10px;
		margin-right: 5px;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		transition: background-color 0.15s ease-in-out;
	}

	.btn-expand {
		background-color: #007bff;
		color: white;
	}

	.btn-expand:hover {
		background-color: #0056b3;
	}

	.btn-delete {
		background-color: #dc3545;
		color: white;
	}

	.btn-delete:hover {
		background-color: #c82333;
	}

	@media screen and (max-width: 600px) {
		.collapsible-table, .collapsible-table tbody, .collapsible-table tr, .collapsible-table td {
			display: block;
			width: 100%;
		}

		.collapsible-table thead {
			display: none;
		}

		.collapsible-table tr {
			margin-bottom: 10px;
			border: 1px solid #dee2e6;
		}

		.collapsible-table td {
			display: flex;
			justify-content: space-between;
			text-align: right;
			padding-left: 50%;
			position: relative;
		}

		.collapsible-table td::before {
			content: attr(data-label);
			position: absolute;
			left: 6px;
			width: 45%;
			text-align: left;
			font-weight: bold;
		}

		.patient-details td {
			padding-left: 10px;
		}

		.patient-details td::before {
			content: none;
		}

		.details-grid {
			grid-template-columns: 1fr;
		}
	}
</style>
