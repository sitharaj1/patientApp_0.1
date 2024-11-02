<script lang="ts">
    export let isNavOpen = false;
    let toggleNav = () => {
        isNavOpen = !isNavOpen;
    }
</script>

<nav class:open={isNavOpen}>
    <button class="nav-toggle" on:click={toggleNav}>
        {isNavOpen ? "✕" : "☰"}
    </button>
    <ul>
        <li>
            <a href="/" 
                >Home</a
            >
        </li>
        <li>
            <a
                href="/search"
                >Search Patients</a
            >
        </li>
        <li>
            <a
                href="/patient"
                >Add Patient</a
            >
        </li>
    </ul>
</nav>
