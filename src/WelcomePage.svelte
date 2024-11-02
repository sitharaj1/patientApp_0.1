<script lang="ts">
    import SideNav from "./SideNav.svelte";
    
    let BASE_URL: string = localStorage.getItem('BASE_URL') || "No URL set";
    let contextURL = "";
	let showAddPatientForm = false;
	let isNavOpen = false;
	let activeSection = "home"; // Default to home/welcome page view

	function toggleNav() {
		isNavOpen = !isNavOpen;
	}

	function setActiveSection(section: string) {
		activeSection = section;
		isNavOpen = false; // Close nav after selection on mobile
	}

 	function isValidUrl(url: string): boolean {
		const urlPattern = new RegExp(
			"^(https?:\\/\\/)?" + // protocol
				"((([a-zA-Z\\d]([a-zA-Z\\d-]*[a-zA-Z\\d])*)\\.)+[a-zA-Z]{2,}|" + // domain name
				"((\\\d{1,3}\\\.){3}\\\d{1,3}))" + // OR ip (v4) address
				"(\\\:\\d+)?(\\\/[-a-zA-Z\\d%_.~+]*)*" + // port and path
				"(\\\?[;&a-zA-Z\\d%_.~+=-]*)?" + // query string
				"(\\\#[-a-zA-Z\\d_]*)?$", // fragment locator
			"i",
		);

		return !!urlPattern.test(url);
	}

	function validateContextURL(): string {
		if (!contextURL) {
			return "Context URL is required.";
		}
		if (!isValidUrl(contextURL)) {
			return "Invalid URL.";
		}
		return "";
	}

    async function setURLInHomePage() {
        const validationError = validateContextURL();
		if (validationError) {
			alert(validationError);
			return;
		}
        BASE_URL = contextURL;
		localStorage.setItem('BASE_URL', BASE_URL);
		alert(`Server URL set to: ${BASE_URL}`);
	}

	async function setServerURL(event: SubmitEvent & { currentTarget: EventTarget & HTMLFormElement; }) {
					throw new Error("Function not implemented.");
		const validationError = validateContextURL();
		if (validationError) {
			alert(validationError);
			return;
		}
		
		try {
			BASE_URL = contextURL;

		} catch (error) {
			console.error("Error setting server URL:", error);
			alert("Failed to set server URL. Please try again.");
		}
	}

	function resetBaseURL() {
		BASE_URL = "";
		contextURL = "";
		localStorage.removeItem('BASE_URL');
		alert('Context URL has been reset');
	}
</script>

<main class:nav-open={isNavOpen}>
	<SideNav isNavOpen={isNavOpen} />
	<form on:submit|preventDefault={setServerURL}>
	<div class="content">
		<table class="main-table">
			<tr>
				<td>
					<form on:submit|preventDefault={setURLInHomePage}>
						<h1 class="text-2xl pb-4">Enter Context URL</h1>
						<input
							type="text"
							placeholder="FHIR Server URL"
							bind:value={contextURL}
							required
						/>
						<button type="submit" class="btn-secondary">Set Context URL</button>
						<button type="button" class="btn-reset" on:click={resetBaseURL}>Reset URL</button>
					</form>
				</td>
			</tr>
		</table>
		<h2>Current Context URL set to : {BASE_URL}</h2>
	</div>
</main>
