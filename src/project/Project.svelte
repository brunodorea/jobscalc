<script>
    import Header from "../parts/Header.svelte"
    import Main from "./Main.svelte"
    import Aside from "../parts/Aside.svelte"
    import { app } from "../store.js"
    import { Calculate } from "../scripts/calculate"
    import { Project } from "../scripts/projects"
    import { navigate } from "svelte-routing"

    /** @type {string|undefined} */
    export let id = undefined

    let isEditing = false
    let projectFound = true
    let name = ''
    /** @type {number|string} */
    let dailyHours = 1
    /** @type {number|string} */
    let totalHours = 1
    let currentId = ''
    let createdAt = new Date()

    let errors = {
        name: '',
        dailyHours: '',
        totalHours: ''
    }

    /** @type {string|null|undefined} */
    let loadedId = null

    /** @param {string|undefined} targetId */
    function syncProject(targetId) {
        if (targetId) {
            const existing = $app.projects.find((/** @type {Project} */ p) => String(p.id) === String(targetId))
            if (existing) {
                isEditing = true
                projectFound = true
                name = existing.name
                dailyHours = existing.dailyHours
                totalHours = existing.totalHours
                currentId = existing.id
                createdAt = existing.createdAt
                $app.currentProject = existing
                return
            } else {
                isEditing = false
                projectFound = false
                return
            }
        }

        // Novo Job (/project)
        isEditing = false
        projectFound = true
        name = ''
        dailyHours = 1
        totalHours = 1
        currentId = crypto.randomUUID()
        createdAt = new Date()
        $app.currentProject = new Project('', 1, 1, currentId, createdAt)
    }

    $: if (id !== loadedId) {
        loadedId = id
        syncProject(id)
    }

    $: tempProject = new Project(name, Number(dailyHours) || 0, Number(totalHours) || 0, currentId, createdAt)
    $: projectValue = new Calculate($app, tempProject).formattedProjectValue
    $: pageTitle = !projectFound ? "Job não encontrado" : (isEditing ? "Editar Job" : "Novo Job")

    function validate() {
        errors = { name: '', dailyHours: '', totalHours: '' }
        let isValid = true

        if (!name || !name.trim()) {
            errors.name = 'O nome do projeto é obrigatório.'
            isValid = false
        }

        if (dailyHours === '' || dailyHours === null || Number(dailyHours) <= 0) {
            errors.dailyHours = 'As horas diárias devem ser maiores que zero.'
            isValid = false
        } else if (Number(dailyHours) > 24) {
            errors.dailyHours = 'As horas diárias não podem ser maiores que 24.'
            isValid = false
        }

        if (totalHours === '' || totalHours === null || Number(totalHours) <= 0) {
            errors.totalHours = 'A estimativa total de horas deve ser maior que zero.'
            isValid = false
        }

        return isValid
    }

    function handleSave() {
        if (!validate()) return

        const updatedProject = new Project(
            name.trim(),
            Number(dailyHours),
            Number(totalHours),
            currentId,
            createdAt
        )

        const existingIndex = $app.projects.findIndex((/** @type {Project} */ p) => String(p.id) === String(currentId))
        if (existingIndex >= 0) {
            const updated = [...$app.projects]
            updated[existingIndex] = updatedProject
            $app.projects = updated
        } else {
            $app.projects = [...$app.projects, updatedProject]
        }

        $app.page = 'home'
        navigate('/')
    }

    function handleCancel() {
        $app.page = 'home'
        navigate('/')
    }

    function handleDelete() {
        if (confirm("Tem certeza que deseja excluir esse projeto?")) {
            $app.projects = $app.projects.filter((/** @type {Project} */ p) => String(p.id) !== String(currentId))
            $app.page = 'home'
            navigate('/')
        }
    }
</script>

{#if !projectFound}
    <div class="bg-gray-100 min-h-screen">
        <Header title={pageTitle} />
        <div class="container animate-up flex flex-col items-center justify-center p-12 max-w-lg mx-auto text-center">
            <div class="bg-white p-8 rounded-lg shadow-sm border border-gray-200 w-full space-y-4">
                <img src="/images/alert-octagon.svg" alt="Aviso" class="w-12 h-12 mx-auto" />
                <h2 class="text-xl font-bold text-gray-700">Projeto não encontrado</h2>
                <p class="text-gray-500 text-sm">O projeto com o ID especificado não existe ou foi removido.</p>
                <button
                    type="button"
                    on:click={() => navigate('/')}
                    class="bg-orange-400 hover:bg-orange-500 text-white font-bold text-xs uppercase px-6 py-3 rounded transition-all inline-block shadow-sm"
                >
                    Voltar para Home
                </button>
            </div>
        </div>
    </div>
{:else}
    <div class="bg-gray-100 min-h-screen">
        <Header title={pageTitle} />
        <div
            class="container animate-up delay-2 flex justify-between p-12 max-w-4xl mx-auto gap-16"
        >
            <div class="w-1/2">
                <Main
                    bind:name
                    bind:dailyHours
                    bind:totalHours
                    {errors}
                    {isEditing}
                    on:save={handleSave}
                    on:cancel={handleCancel}
                    on:delete={handleDelete}
                />
            </div>    

            <div class="flex-grow-0 text-center">
                <Aside>
                    <img src="/images/money-color.svg" alt="Imagem de Dinheiro" class="mx-auto" />
                    <p class="mt-8 text-gray-600">
                        O valor do projeto ficou em <strong>{projectValue}</strong>
                    </p>
                </Aside>
            </div>
        </div>
    </div>
{/if}
