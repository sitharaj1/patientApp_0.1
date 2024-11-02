<script lang="ts">
    import AddPatientForm from "./AddPatientForm.svelte";
    import PatientList from "./PatientList.svelte";
    import WelcomePage from "./WelcomePage.svelte";
    
    import { Router, Link, Route } from "svelte-routing";
</script>
<Router>
    <main>
        <div class="w-full mx-auto my-10">
            <Route path="/">
                <WelcomePage></WelcomePage>
            </Route>
            <Route path="/search">
                <PatientList></PatientList>
            </Route>
            <Route path="/patient">
                <AddPatientForm></AddPatientForm>
                <Route path="/patient/:id" component={AddPatientForm}></Route>
            </Route>
            
        </div>
    </main>
</Router>
