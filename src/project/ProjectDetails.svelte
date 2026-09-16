<script>
    import Header from "../parts/Header.svelte"
    import Aside from "../parts/Aside.svelte"
    import { app } from "../store.js"
    import { Calculate } from "../scripts/calculate.js"
    import { dayjs } from "../scripts/dayjs.js"
    import { navigate } from "svelte-routing"
    import { Project } from "../scripts/projects.js"

    /** @type {string|undefined} */
    export let id = undefined

    /** @type {Project|null} */
    let project = null
    let projectFound = true
    /** @type {string|null|undefined} */
    let loadedId = null

    /** @param {string|undefined} targetId */
    function syncProject(targetId) {
        if (targetId) {
            const existing = $app.projects.find((/** @type {Project} */ p) => String(p.id) === String(targetId))
            if (existing) {
                project = existing
                projectFound = true
                $app.currentProject = existing
                return
            }
        }
        project = null
        projectFound = false
    }

    $: if (id !== loadedId) {
        loadedId = id
        syncProject(id)
    }

    $: projectValue = project ? new Calculate($app, project).formattedProjectValue : 'R$ 0,00'
    $: hourValue = new Calculate($app).formattedValueHour

    function handleEdit() {
        if (project) {
            navigate(`/project/${project.id}/edit`)
        }
    }

    function handleDelete() {
        if (!project) return
        if (confirm("Tem certeza que deseja excluir esse projeto?")) {
            $app.projects = $app.projects.filter((/** @type {Project} */ p) => String(p.id) !== String(project?.id))
            $app.page = 'home'
            navigate('/')
        }
    }
</script>

{#if !projectFound || !project}
    <div class="bg-gray-100 min-h-screen">
        <Header title="Projeto não encontrado" />
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
        <Header title="Informações do projeto" />
        <div
            class="container animate-up delay-2 flex justify-between p-12 max-w-4xl mx-auto gap-16"
        >
            <div class="w-1/2 space-y-6">
                <!-- Informações Principais -->
                <div class="bg-white border border-gray-200 rounded p-6 shadow-sm">
                    <div class="flex items-start justify-between border-b pb-4 mb-4 gap-4">
                        <div>
                            <span class="text-xs font-bold text-gray-400 uppercase tracking-wider">Projeto</span>
                            <h2 class="text-2xl font-bold text-gray-700 mt-1">{project.name}</h2>
                        </div>
                        <div class={`px-3 py-1 rounded-full text-xs font-bold uppercase tracking-wider flex items-center gap-1.5 ${project.status === "encerrado" ? "bg-red-100 text-red-600" : "bg-green-100 text-green-600"}`}>
                            <span class={`w-2 h-2 rounded-full ${project.status === "encerrado" ? "bg-red-500" : "bg-green-500"}`}></span>
                            {project.status === "encerrado" ? "Encerrado" : "Em andamento"}
                        </div>
                    </div>

                    <!-- Métricas do Projeto -->
                    <div class="grid grid-cols-2 gap-4">
                        <div class="p-3 bg-gray-50 rounded border border-gray-100">
                            <span class="text-xs font-bold text-gray-400 uppercase">Horas diárias</span>
                            <p class="text-lg font-bold text-gray-700 mt-1">
                                {project.dailyHours} {project.dailyHours === 1 ? 'hora' : 'horas'} / dia
                            </p>
                        </div>

                        <div class="p-3 bg-gray-50 rounded border border-gray-100">
                            <span class="text-xs font-bold text-gray-400 uppercase">Estimativa total</span>
                            <p class="text-lg font-bold text-gray-700 mt-1">
                                {project.totalHours} {project.totalHours === 1 ? 'hora' : 'horas'}
                            </p>
                        </div>

                        <div class="p-3 bg-gray-50 rounded border border-gray-100">
                            <span class="text-xs font-bold text-gray-400 uppercase">Prazo restante</span>
                            {#if project.status === "em andamento"}
                                <p class="text-lg font-bold text-gray-700 mt-1">
                                    {project.remainingDays} {project.remainingDays === 1 ? "dia" : "dias"}
                                </p>
                            {:else}
                                <p class="text-lg font-bold text-red-600 mt-1">
                                    Prazo esgotado
                                </p>
                            {/if}
                        </div>

                        <div class="p-3 bg-gray-50 rounded border border-gray-100">
                            <span class="text-xs font-bold text-gray-400 uppercase">Previsão de entrega</span>
                            <p class="text-lg font-bold text-gray-700 mt-1">
                                {project.deadline ? project.deadline.format('DD/MM/YYYY') : '-'}
                            </p>
                        </div>
                    </div>

                    <div class="mt-4 pt-4 border-t border-gray-100 flex justify-between text-xs text-gray-400">
                        <span>Criado em: {dayjs(project.createdAt).format('DD/MM/YYYY')}</span>
                        <span>Valor/hora base: {hourValue}</span>
                    </div>
                </div>

                <!-- Ações -->
                <div class="flex items-center justify-between pt-2">
                    <div class="flex gap-3">
                        <button
                            type="button"
                            on:click={handleEdit}
                            class="bg-orange-400 hover:bg-orange-500 text-white font-bold text-xs uppercase px-6 py-3 rounded transition-all shadow-sm flex items-center gap-2"
                        >
                            <img src="/images/edit-24.svg" alt="" class="w-4 h-4 brightness-0 invert" />
                            <span>Editar projeto</span>
                        </button>
                        <button
                            type="button"
                            on:click={() => navigate('/')}
                            class="bg-gray-200 hover:bg-gray-300 text-gray-700 font-bold text-xs uppercase px-6 py-3 rounded transition-all"
                        >
                            Voltar
                        </button>
                    </div>

                    <button
                        type="button"
                        on:click={handleDelete}
                        class="flex items-center gap-2 border border-gray-300 hover:border-red-400 hover:bg-red-50 px-4 py-3 rounded transition-all text-gray-600 hover:text-red-600 text-xs font-semibold"
                        title="Excluir projeto"
                    >
                        <img src="/images/trash-24.svg" alt="Excluir" class="w-4 h-4" />
                        <span>Excluir projeto</span>
                    </button>
                </div>
            </div>    

            <div class="flex-grow-0 text-center">
                <Aside>
                    <img src="/images/money-color.svg" alt="Imagem de Dinheiro" class="mx-auto" />
                    <p class="mt-8 text-gray-600">
                        O valor deste projeto é de <strong>{projectValue}</strong>
                    </p>
                </Aside>
            </div>
        </div>
    </div>
{/if}
