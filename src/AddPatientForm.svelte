<script lang="ts">
	let isNavOpen = false;
	import SideNav from "./SideNav.svelte";

	let activeSection = "home"; // Default to home/welcome page view
	let showAddPatientForm = false;
	let BASE_URL = localStorage.getItem('BASE_URL') || "";
	let dateError = '';

	function toggleNav() {
		isNavOpen = !isNavOpen;
	}

	function setActiveSection(section: string) {
		activeSection = section;
		isNavOpen = false; // Close nav after selection on mobile
	}

	let newPatient = {
		firstName: "",
		lastName: "",
		gender: "",
		birthDate: "",
		phoneNumber: "",
	};

	// Add this function to get the current date in YYYY-MM-DD format
	function getCurrentDate(): string {
		return new Date().toISOString().split("T")[0];
	}

	function isValidName(name: string): boolean {
		return /^[a-zA-Z\s-]{1,50}$/.test(name);
	}

	function validateNewPatientForm(): string {
		if (!newPatient.firstName || !newPatient.lastName) {
			return "First name and last name are required.";
		}
		if (
			!isValidName(newPatient.firstName) ||
			!isValidName(newPatient.lastName)
		) {
			return "Invalid First name/Last name. Please use only letters, spaces, and hyphens (max 50 characters).";
		}
		if (!newPatient.gender) {
			return "Please select a gender.";
		}
		if (!newPatient.birthDate) {
			return "Birth date is required.";
		}

		const currentDate = getCurrentDate();
		if (newPatient.birthDate > currentDate) {
			return "Birth date cannot be in the future.";
		}

		if (!newPatient.phoneNumber) {
			return "Phone number is required.";
		}
		if (!/^\d{5,15}$/.test(newPatient.phoneNumber)) {
			return "Invalid phone number. Please enter 5-15 digits.";
		}
		return "";
	}

	function validateDate(date: string) {
		if (!date) {
			dateError = 'Birth date is required';
			return false;
		}

		const birthDate = new Date(date);
		const today = new Date();
		const minDate = new Date('1900-01-01');
		
		if (isNaN(birthDate.getTime())) {
			dateError = 'Invalid date format';
			return false;
		}
		
		if (birthDate < minDate) {
			dateError = 'Date cannot be before 1900-01-01';
			return false;
		}
		
		if (birthDate > today) {
			dateError = 'Date cannot be in the future';
			return false;
		}

		dateError = '';
		return true;
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
					given: [newPatient.firstName],
				},
			],
			gender: newPatient.gender,
			birthDate: newPatient.birthDate,
			telecom: [
				{
					system: "phone",
					value: newPatient.phoneNumber,
					use: "mobile",
				},
			],
		};

		try {
			const response = await fetch(BASE_URL+"/Patient", {
				method: "POST",
				headers: {
					"Content-Type": "application/json",
					// Add any necessary authentication headers here
				},
				body: JSON.stringify(patientResource),
			});

			if (!response.ok) {
				throw new Error(`HTTP error! status: ${response.status}`);
			}

			const result = await response.json();
			alert(`Patient added successfully with ID: ${result.id}`);
			newPatient = {
				firstName: "",
				lastName: "",
				gender: "",
				birthDate: "",
				phoneNumber: "",
			};
		} catch (error) {
			console.error("Error adding patient:", error);
			alert("Failed to add patient. Please try again.");
		}
	}
</script>

<main class:nav-open={isNavOpen}>
	<SideNav isNavOpen={isNavOpen} />
		<h1 class="text-2xl pb-4">Add New Patient</h1>
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
			<div class="input-group">
				<input
					type="date"
					bind:value={newPatient.birthDate}
					min="1900-01-01"
					max={getCurrentDate()}
					on:blur={() => validateDate(newPatient.birthDate)}
					class:error={dateError}
					required
				/>
				{#if dateError}
					<span class="error-message">{dateError}</span>
				{/if}
			</div>
			<input
				type="tel"
				placeholder="Phone Number"
				bind:value={newPatient.phoneNumber}
				required
			/>
		<button type="submit" class="btn-secondary">Add Patient</button>
	</form>
</main>

<style>
	main {
		padding: 20px;
		text-align: center;
	}

	form {
		display: flex;
		flex-direction: column;
		gap: 10px;
		max-width: 400px;
		margin: 0 auto;
	}

	input,
	select {
		padding: 8px 12px;
		font-size: 14px;
		border: 1px solid #ccc;
		border-radius: 4px;
	}

	.btn-secondary {
		padding: 10px 20px;
		color: white;
		background-color: #008cba;
		border: none;
		border-radius: 4px;
		cursor: pointer;
		font-size: 16px;
	}

	.input-group {
		display: flex;
		flex-direction: column;
		align-items: start;
		gap: 4px;
	}

	.error {
		border-color: #dc3545;
	}

	.error-message {
		color: #dc3545;
		font-size: 0.875rem;
		text-align: left;
	}
</style>