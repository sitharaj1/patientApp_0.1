<script lang="ts">
	import { slide } from "svelte/transition";
	import SideNav from "./SideNav.svelte";

	let BASE_URL = localStorage.getItem('BASE_URL') || "";
	let pageName = "Search Patients";
	let pageDescription =
		"This page displays patient details from Medblocks FHIR Server. ";
	let getAllPatientsDescription =
		"Click here to see all patients on the Server";
	let showResults = false;
	let patientData = { firstName: "", lastName: "", uid: "", gender: "" };
	let errorMessage = "";
	let patients: any[] = [];
	let patientCount = 0;

	let newPatient = {
		firstName: "",
		lastName: "",
		gender: "",
		birthDate: "",
		phoneNumber: "",
	};
	let showAddPatientForm = false;

	let isNavOpen = false;
	let activeSection = "search"; // Default to search view

	let expandedPatientId: string | null = null;

	let editingPatient: any = null;
	let editForm = {
		firstName: "",
		lastName: "",
		gender: "",
		birthDate: "",
		phoneNumber: ""
	};

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
			const response = await fetch(BASE_URL+"/Patient");
			const data = await response.json();
			patients =
				data.entry?.map((entry: { resource: any }) => entry.resource) || [];
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

		let queryString = BASE_URL+"/Patient";

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
			patients = data.entry?.map(
				(entry: { resource: any }) => entry.resource
			) || [data];
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

	async function deletePatient(patientId: string) {
		if (
			!confirm(`Are you sure you want to delete patient with ID ${patientId}?`)
		) {
			return;
		}

		try {
			const response = await fetch(`${BASE_URL}/Patient/${patientId}`, {
				method: "DELETE",
				headers: {
					"Content-Type": "application/json",
					// Add any necessary authentication headers here
				},
			});

			if (!response.ok) {
				console.error(`HTTP error! status: `);
				throw new Error(`HTTP error! status: ${response.status}`);
			}

			// Remove the deleted patient from the patients array
			patients = (patients as any[]).filter(
				(patient: { id: string }) => patient.id !== patientId
			);

			alert(`Patient with ID ${patientId} has been successfully deleted.`);
		} catch (error) {
			console.error("Error deleting patient:", error);
			alert(`Failed to delete patient with ID ${patientId}. Please try again.`);
		}
	}

	function startEditing(patient: any) {
		editingPatient = patient;
		editForm = {
			firstName: patient.name?.[0]?.given?.[0] || "",
			lastName: patient.name?.[0]?.family || "",
			gender: patient.gender || "",
			birthDate: patient.birthDate || "",
			phoneNumber: patient.telecom?.[0]?.value || ""
		};
	}

	async function updatePatient() {
		try {
			const updatedResource = {
				resourceType: "Patient",
				id: editingPatient.id,
				name: [{
					use: "official",
					family: editForm.lastName,
					given: [editForm.firstName]
				}],
				gender: editForm.gender,
				birthDate: editForm.birthDate,
				telecom: [{
					system: "phone",
					value: editForm.phoneNumber,
					use: "mobile"
				}]
			};

			const response = await fetch(`${BASE_URL}/Patient/${editingPatient.id}`, {
				method: 'PUT',
				headers: {
					'Content-Type': 'application/json',
				},
				body: JSON.stringify(updatedResource)
			});

			if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);

			const result = await response.json();
			
			// Update the patients array
			patients = patients.map(p => p.id === editingPatient.id ? result : p);
			
			editingPatient = null;
			alert('Patient updated successfully');
		} catch (error) {
			console.error('Error updating patient:', error);
			alert('Failed to update patient');
		}
	}

	// Validation functions
	function isValidName(name: string): boolean {
		return /^[a-zA-Z\s-]{1,50}$/.test(name);
	}

	function isValidUUID(uuid: string): boolean {
		const uuidRegex =
			/^[0-9a-fA-F]{8}-[0-9a-fA-F]{4}-[1-5][0-9a-fA-F]{3}-[89abAB][0-9a-fA-F]{3}-[0-9a-fA-F]{12}$/;
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
</script>

<main class:nav-open={isNavOpen}>
	<SideNav isNavOpen={isNavOpen} />
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
						<button class="btn-primary" on:click={fetchAllPatients}
							>Get All Patients</button
						>
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
							class:invalid={patientData.firstName &&
								!isValidName(patientData.firstName)}
						/>
						<input
							type="text"
							placeholder="Last Name"
							bind:value={patientData.lastName}
							class:invalid={patientData.lastName &&
								!isValidName(patientData.lastName)}
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
							<button type="submit" class="btn-secondary"
								>Search Patients</button
							>
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
											<tr
												class="patient-row"
												class:expanded={expandedPatientId === patient.id}
											>
												<td
													>{patient.name?.[0]?.given?.join(" ")}
													{patient.name?.[0]?.family}</td
												>
												<td>{patient.gender}</td>
												<td>{patient.id}</td>
												<td>
													<button
														class="btn-expand"
														on:click={() => togglePatientDetails(patient.id)}
													>
														{expandedPatientId === patient.id ? "▲" : "▼"}
													</button>
													<button
														class="btn-edit"
														on:click={() => startEditing(patient)}
														>Edit</button>
													<button
														class="btn-delete"
														on:click={() => deletePatient(patient.id)}
														>Delete</button
													>
												</td>
											</tr>
											{#if expandedPatientId === patient.id}
												<tr class="patient-details" transition:slide|local>
													<td colspan="4">
														<div class="details-grid">
															<div>
																<strong>Birth Date:</strong>
																{patient.birthDate || "N/A"}
															</div>
															<div>
																<strong>Phone:</strong>
																{patient.telecom?.[0]?.value || "N/A"}
															</div>
															<div>
																<strong>Address:</strong>
																{patient.address?.[0]?.line?.join(", ") ||
																	"N/A"}
															</div>
															<div>
																<strong>City:</strong>
																{patient.address?.[0]?.city || "N/A"}
															</div>
															<div>
																<strong>State:</strong>
																{patient.address?.[0]?.state || "N/A"}
															</div>
															<div>
																<strong>Country:</strong>
																{patient.address?.[0]?.country || "N/A"}
															</div>
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
				</tr>{/if}

			{#if errorMessage}
				<div class="error-message">{errorMessage}</div>
			{/if}
		</table>
	</div>
</main>

{#if editingPatient}
	<div class="edit-modal">
		<div class="edit-content">
			<h2>Edit Patient</h2>
			<form on:submit|preventDefault={updatePatient}>
				<input type="text" bind:value={editForm.firstName} placeholder="First Name" required>
				<input type="text" bind:value={editForm.lastName} placeholder="Last Name" required>
				<select bind:value={editForm.gender} required>
					<option value="">Select gender</option>
					<option value="male">Male</option>
					<option value="female">Female</option>
					<option value="other">Other</option>
				</select>
				<input type="date" bind:value={editForm.birthDate} required>
				<input type="tel" bind:value={editForm.phoneNumber} placeholder="Phone Number" required>
				<div class="button-group">
					<button type="submit" class="btn-save">Save</button>
					<button type="button" class="btn-cancel" on:click={() => editingPatient = null}>
						Cancel
					</button>
				</div>
			</form>
		</div>
	</div>
{/if}
