<script lang="ts">
    import { onMount } from "svelte";
    import { ROOT_URL } from "$lib";

    type Plant = {
        id: number;
        name: string;
        slug: string;
        category: string;
        description: string;
        ideal_soil_moisture_min: number;
        ideal_soil_moisture_max: number;
        watering_note: string;
        is_active: boolean;
    };

    type PlantListResponse = {
        count: number;
        total_pages: number;
        current_page: number;
        page_size: number;
        results: Plant[];
    };
    type ControlConfig = {
        id: number;
        selected_plant: Plant | null;
        auto_mode: boolean;
        manual_override: boolean;
        motor_enabled: boolean;
        updated_at: string;
    };

    let plants: Plant[] = $state([]);
    let config: ControlConfig | null = $state(null);
    let isLoading = $state(true);
    let isSaving = $state(false);
    let error = $state("");
    let success = $state("");

    function getToken() {
        return localStorage.getItem("access");
    }

    async function fetchPlants() {
        const token = getToken();
        if (!token) {
            error = "Token topilmadi";
            return;
        }
    
        const res = await fetch(`${ROOT_URL}/plants/plants/`, {
            headers: {
                Authorization: `Token ${token}`,
            },
        });
    
        if (!res.ok) {
            throw new Error("Plants ma'lumotlarini olishda xatolik");
        }
    
        const json = await res.json();
        plants = json.results || [];
    }

    async function fetchConfig() {
        const token = getToken();
        if (!token) {
            error = "Token topilmadi";
            return;
        }

        const res = await fetch(`${ROOT_URL}/plants/control/config/`, {
            headers: {
                Authorization: `Token ${token}`,
            },
        });

        if (!res.ok) {
            throw new Error("Control config ma'lumotlarini olishda xatolik");
        }

        config = await res.json();
    }

    async function loadData() {
        isLoading = true;
        error = "";

        try {
            await Promise.all([fetchPlants(), fetchConfig()]);
        } catch (err: any) {
            error = err.message || "Xatolik yuz berdi";
        } finally {
            isLoading = false;
        }
    }

    async function selectPlant(plantId: number) {
        const token = getToken();
        if (!token || !config) return;

        isSaving = true;
        error = "";
        success = "";

        try {
            const res = await fetch(`${ROOT_URL}/plants/control/config/`, {
                method: "PATCH",
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Token ${token}`,
                },
                body: JSON.stringify({
                    selected_plant_id: plantId,
                }),
            });

            if (!res.ok) {
                throw new Error("O'simlikni tanlashda xatolik");
            }

            config = await res.json();
            success = "O'simlik muvaffaqiyatli tanlandi";
        } catch (err: any) {
            error = err.message || "Saqlashda xatolik";
        } finally {
            isSaving = false;
            setTimeout(() => {
                success = "";
            }, 2500);
        }
    }

    async function toggleAutoMode() {
        const token = getToken();
        if (!token || !config) return;
    
        isSaving = true;
        error = "";
        success = "";
    
        try {
            const nextAutoMode = !config.auto_mode;
    
            const res = await fetch(`${ROOT_URL}/plants/control/config/`, {
                method: "PATCH",
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Token ${token}`,
                },
                body: JSON.stringify({
                    auto_mode: nextAutoMode,
                }),
            });
    
            if (!res.ok) {
                throw new Error("Auto mode ni yangilashda xatolik");
            }
    
            config = await res.json();
            success = config.auto_mode
                ? "Auto mode yoqildi"
                : "Auto mode o'chirildi va motor to'xtatildi";
        } catch (err: any) {
            error = err.message || "Yangilashda xatolik";
        } finally {
            isSaving = false;
            setTimeout(() => {
                success = "";
            }, 2500);
        }
    }

    function isSelectedPlant(plantId: number) {
        return config?.selected_plant?.id === plantId;
    }

    onMount(async () => {
        await loadData();
    });
</script>

<div class="min-h-screen bg-black text-white">
    <div class="w-[80%] fixed right-0 max-w-screen-2xl mx-auto px-4 sm:px-6 lg:px-8 pt-16 pb-10 space-y-6">
        <div class="flex flex-col gap-4 md:flex-row md:items-center md:justify-between">
            <div>
                <h1 class="text-3xl font-bold text-white">Control Page</h1>
                <p class="text-sm text-teal-500 mt-1">
                    O'simlik tanlash va avtomatik sug'orish sozlamalari
                </p>
            </div>

            {#if config}
                <div class="flex items-center gap-3">
                    <span class="text-sm text-slate-400">
                        Auto mode:
                    </span>
                    <button
                        onclick={toggleAutoMode}
                        disabled={isSaving}
                        class={`px-4 py-2 rounded-xl border transition-all duration-300 font-medium
                        ${
                            config.auto_mode
                                ? "bg-teal-600/20 border-teal-500 text-teal-400"
                                : "bg-slate-800 border-slate-600 text-slate-300"
                        }`}
                    >
                        {config.auto_mode ? "Yoqilgan" : "O'chirilgan"}
                    </button>
                </div>
            {/if}
        </div>

        {#if error}
            <div class="rounded-2xl border border-red-500/30 bg-red-500/10 px-4 py-3 text-red-300">
                {error}
            </div>
        {/if}

        {#if success}
            <div class="rounded-2xl border border-teal-500/30 bg-teal-500/10 px-4 py-3 text-teal-300">
                {success}
            </div>
        {/if}

        {#if isLoading}
            <div class="grid grid-cols-3 gap-5">
                {#each Array(6) as _}
                    <div class="h-48 rounded-2xl border border-slate-800 bg-slate-900/40 animate-pulse"></div>
                {/each}
            </div>
        {:else}
        <div class="grid grid-cols-1 xl:grid-cols-3 gap-6">
            <div class="xl:col-span-2 space-y-6">
                    <div class="rounded-2xl border border-teal-700 bg-transparent backdrop-blur-md overflow-hidden">
                        <div class="px-6 py-4 border-b border-teal-700">
                            <h2 class="text-lg font-semibold text-slate-100">
                                O'simliklar ro'yxati
                            </h2>
                            <p class="text-sm text-teal-500 mt-1">
                                O'zingizga kerakli o'simlikni tanlang
                            </p>
                        </div>

                        <div class="p-4 sm:p-5 grid grid-cols-1 md:grid-cols-2 gap-4">
                            {#each plants as plant}
                                <button
                                    onclick={() => selectPlant(plant.id)}
                                    disabled={isSaving}
                                    class={`text-left rounded-2xl border p-4 transition-all duration-300
                                    ${
                                        isSelectedPlant(plant.id)
                                            ? "border-teal-400 bg-teal-500/10 shadow-lg shadow-teal-500/10"
                                            : "border-slate-700 bg-slate-900/30 hover:border-teal-600 hover:bg-teal-500/5"
                                    }`}
                                >
                                    <div class="flex items-start justify-between gap-3">
                                        <div>
                                            <h3 class="text-lg font-semibold text-white">
                                                {plant.name}
                                            </h3>
                                            <p class="text-xs text-teal-500 mt-1">
                                                {plant.category || "Kategoriya ko'rsatilmagan"}
                                            </p>
                                        </div>

                                        {#if isSelectedPlant(plant.id)}
                                            <span class="text-xs px-2 py-1 rounded-full bg-teal-500/20 text-teal-300 border border-teal-500/30">
                                                Tanlangan
                                            </span>
                                        {/if}
                                    </div>

                                    <p class="text-sm text-slate-400 mt-3 line-clamp-3">
                                        {plant.description || "Ma'lumot kiritilmagan"}
                                    </p>

                                    <div class="mt-4 pt-4 border-t border-slate-800">
                                        <div class="flex items-center justify-between text-sm">
                                            <span class="text-slate-400">Tuproq namligi</span>
                                            <span class="text-teal-400 font-medium">
                                                {plant.ideal_soil_moisture_min}% - {plant.ideal_soil_moisture_max}%
                                            </span>
                                        </div>
                                    </div>
                                </button>
                            {/each}
                        </div>
                    </div>
                </div>

                <div class="space-y-6">
                    <div class="rounded-2xl border border-teal-700 bg-transparent backdrop-blur-md overflow-hidden">
                        <div class="px-6 py-4 border-b border-teal-700">
                            <h2 class="text-lg font-semibold text-slate-100">
                                Tanlangan o'simlik
                            </h2>
                        </div>

                        <div class="p-5">
                            {#if config?.selected_plant}
                                <div class="space-y-4">
                                    <div>
                                        <h3 class="text-2xl font-bold text-white">
                                            {config.selected_plant.name}
                                        </h3>
                                        <p class="text-sm text-teal-500 mt-1">
                                            {config.selected_plant.category || "Kategoriya ko'rsatilmagan"}
                                        </p>
                                    </div>

                                    <div class="rounded-xl border border-slate-700 bg-slate-900/40 p-4">
                                        <p class="text-xs uppercase tracking-wide text-slate-500 mb-2">
                                            Tavsif
                                        </p>
                                        <p class="text-sm text-slate-300 leading-6">
                                            {config.selected_plant.description || "Tavsif mavjud emas"}
                                        </p>
                                    </div>

                                    <div class="rounded-xl border border-slate-700 bg-slate-900/40 p-4">
                                        <p class="text-xs uppercase tracking-wide text-slate-500 mb-2">
                                            Ideal tuproq namligi
                                        </p>
                                        <p class="text-xl font-semibold text-teal-400">
                                            {config.selected_plant.ideal_soil_moisture_min}% - {config.selected_plant.ideal_soil_moisture_max}%
                                        </p>
                                    </div>

                                    <div class="rounded-xl border border-slate-700 bg-slate-900/40 p-4">
                                        <p class="text-xs uppercase tracking-wide text-slate-500 mb-2">
                                            Sug'orish tavsiyasi
                                        </p>
                                        <p class="text-sm text-slate-300 leading-6">
                                            {config.selected_plant.watering_note || "Tavsiya yozilmagan"}
                                        </p>
                                    </div>
                                </div>
                            {:else}
                                <div class="rounded-xl border border-dashed border-slate-700 p-6 text-center">
                                    <p class="text-slate-400">
                                        Hali o'simlik tanlanmagan
                                    </p>
                                    <p class="text-sm text-teal-500 mt-2">
                                        Chap tomondan biror o'simlikni tanlang
                                    </p>
                                </div>
                            {/if}
                        </div>
                    </div>

                    <div class="rounded-2xl border border-teal-700 bg-transparent backdrop-blur-md overflow-hidden">
                        <div class="px-6 py-4 border-b border-teal-700">
                            <h2 class="text-lg font-semibold text-slate-100">
                                Tizim holati
                            </h2>
                        </div>

                        <div class="p-5 space-y-4">
                            <div class="flex items-center justify-between">
                                <span class="text-slate-400">Auto mode</span>
                                <span class={config?.auto_mode ? "text-teal-400" : "text-red-400"}>
                                    {config?.auto_mode ? "Yoqilgan" : "O'chirilgan"}
                                </span>
                            </div>

                            <div class="flex items-center justify-between">
                                <span class="text-slate-400">Motor boshqaruvi</span>
                                <span class={config?.motor_enabled ? "text-teal-400" : "text-red-400"}>
                                    {config?.motor_enabled ? "Faol" : "Nofaol"}
                                </span>
                            </div>

                            <div class="flex items-center justify-between">
                                <span class="text-slate-400">Manual override</span>
                                <span class={config?.manual_override ? "text-yellow-400" : "text-slate-400"}>
                                    {config?.manual_override ? "Yoqilgan" : "O'chirilgan"}
                                </span>
                            </div>

                            {#if config?.updated_at}
                                <div class="pt-3 border-t border-slate-800">
                                    <p class="text-xs text-slate-500">
                                        Oxirgi yangilanish:
                                    </p>
                                    <p class="text-sm text-slate-300 mt-1">
                                        {new Date(config.updated_at).toLocaleString()}
                                    </p>
                                </div>
                            {/if}
                        </div>
                    </div>
                </div>
            </div>
        {/if}
    </div>
</div>