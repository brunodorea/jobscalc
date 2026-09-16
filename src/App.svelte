<script>
    import "./app.css"
    import { Router, Route } from "svelte-routing"
    import Home from "./home/Home.svelte"
    import Profile from "./profile/Profile.svelte"
    import Project from "./project/Project.svelte"
    import ProjectDetails from "./project/ProjectDetails.svelte"

    export let url = ""
</script>

<Router {url}>
    <Route path="/" component={Home} />
    <Route path="/home" component={Home} />
    <Route path="/Home" component={Home} />
    <Route path="/profile" component={Profile} />
    <Route path="/Profile" component={Profile} />
    <Route path="/project/new" component={Project} />
    <Route path="/Project/new" component={Project} />
    <Route path="/project/:id/edit" component={Project} />
    <Route path="/Project/:id/edit" component={Project} />
    <Route path="/project/:id" component={ProjectDetails} />
    <Route path="/Project/:id" component={ProjectDetails} />
    <Route path="/project" component={Project} />
    <Route path="/Project" component={Project} />
    <Route component={Home} />
</Router>
