<script>
    import "./app.css"
    import { Router, Route } from "svelte-routing"
    import Home from "./home/Home.svelte"
    import Profile from "./profile/Profile.svelte"
    import Project from "./project/Project.svelte"

    export let url = ""
</script>

<Router {url}>
    <Route path="/" component={Home} />
    <Route path="/home" component={Home} />
    <Route path="/Home" component={Home} />
    <Route path="/profile" component={Profile} />
    <Route path="/Profile" component={Profile} />
    <Route path="/project" component={Project} />
    <Route path="/Project" component={Project} />
    <Route path="/project/:id" component={Project} />
    <Route path="/Project/:id" component={Project} />
    <Route component={Home} />
</Router>
