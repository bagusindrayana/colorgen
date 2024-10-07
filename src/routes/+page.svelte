<script lang="ts">
    import { onMount } from "svelte";
    import { page } from "$app/stores";
    import "../app.css";
    import { browser } from "$app/environment";

    let paletteSize = 4;
    let seed = Math.floor(Math.random() * 1000000);
    let palette: string[] = [];

    // $: updatePalette(paletteSize, seed);

    function generateRandomColor(seed: number) {
        const x = Math.sin(seed) * 10000;
        return (
            "#" +
            Math.floor((x - Math.floor(x)) * 16777215)
                .toString(16)
                .padStart(6, "0")
        );
    }

    function generatePalette(count: number, seed: number) {
        return Array.from({ length: count }, (_, i) =>
            generateRandomColor(seed + i),
        );
    }

    function checkQuery() {
        const url = new URL(window.location.href);
        const curSize = url.searchParams.get("size");
        const curSeed = url.searchParams.get("seed");
        if (curSize && curSeed) {
            seed = parseInt(curSeed);
            paletteSize = parseInt(curSize);
            updatePalette(parseInt(curSize), parseInt(curSeed));
        } else {
            regeneratePalette();
        }
    }

    function updatePalette(size: number, seedValue: number) {
        palette = generatePalette(size, seedValue);
        updateURL(size, seedValue);
    }

    function regeneratePalette() {
        seed = Math.floor(Math.random() * 1000000);
        updatePalette(paletteSize, seed);
    }

    function copyToClipboard(color: string) {
        navigator.clipboard.writeText(color).then(
            () => {
                alert(`${color} has been copied to your clipboard.`);
            },
            (err) => {
                console.error("Could not copy text: ", err);
            },
        );
    }

    function updateURL(size: number, seedValue: number) {
        if (!browser) {
            return;
        }
        const url = new URL(window.location.href);
        url.searchParams.set("size", size.toString());
        url.searchParams.set("seed", seedValue.toString());
        window.history.pushState(null, "", url);
    }

    function sharePalette() {
        const url = `${window.location.origin}${window.location.pathname}?size=${paletteSize}&seed=${seed}`;
        navigator.clipboard.writeText(url).then(
            () => {
                alert("URL copied! Share this URL to recreate this palette.");
            },
            (err) => {
                console.error("Could not copy URL: ", err);
            },
        );
    }

    onMount(() => {
        // const params = new URLSearchParams($page.url.searchParams);
        // const sizeParam = params.get("size");
        // const seedParam = params.get("seed");
        // if (sizeParam) paletteSize = parseInt(sizeParam, 10);
        // if (seedParam) seed = parseInt(seedParam, 10);
        // updatePalette(paletteSize, seed);

        checkQuery();
    });
</script>
<svlete:head>
    <title>
        ColorGen - Color Palette Generator
    </title>
    <meta name="description" content="Color Palette Generator">
</svlete:head>
<div
    class="min-h-screen bg-gray-100 flex flex-col items-center justify-center p-4"
>
    <div class="bg-white rounded-lg shadow-md w-full max-w-4xl p-6">
        <h1 class="text-3xl font-bold mb-6 text-center">
            Color Palette Generator
        </h1>
        <div
            class="flex flex-col sm:flex-row justify-between items-center mb-6 gap-4"
        >
            <select
                title="How many colors?"
                bind:value={paletteSize}
                on:change={() => updatePalette(paletteSize, seed)}
                class="w-full sm:w-[180px] p-2 border border-gray-300 rounded-md"
            >
                <option value={2}>2 Colors</option>
                <option value={4}>4 Colors</option>
                <option value={6}>6 Colors</option>
                <option value={8}>8 Colors</option>
            </select>
            <input
                title="Seed"
                type="number"
                bind:value={seed}
                on:input={() => updatePalette(paletteSize, seed)}
                placeholder="Enter seed"
                class="w-full sm:w-[180px] p-2 border border-gray-300 rounded-md"
            />
            <button
                on:click={regeneratePalette}
                class="w-full sm:w-auto bg-blue-500 text-white px-4 py-2 rounded-md hover:bg-blue-600 transition-colors"
            >
                🔄 New Palette
            </button>
            <button
                on:click={sharePalette}
                class="w-full sm:w-auto bg-gray-200 text-gray-800 px-4 py-2 rounded-md hover:bg-gray-300 transition-colors"
            >
                📤 Share
            </button>
        </div>
        <div
            class={`grid gap-4 mb-6 ${
                paletteSize === 2
                    ? "grid-cols-1 sm:grid-cols-2"
                    : paletteSize === 4
                      ? "grid-cols-2 sm:grid-cols-4"
                      : paletteSize === 6
                        ? "grid-cols-2 sm:grid-cols-3"
                        : "grid-cols-2 sm:grid-cols-4"
            }`}
        >
            {#each palette as color, index (index)}
                <div class="flex flex-col items-center">
                    <div
                        class="w-full aspect-square rounded-lg shadow-md cursor-pointer transition-transform hover:scale-105"
                        style="background-color: {color};"
                        on:click={() => copyToClipboard(color)}
                    ></div>
                    <button
                        class="mt-2 text-sm font-mono bg-white px-2 py-1 rounded shadow-sm hover:bg-gray-100 transition-colors"
                        on:click={() => copyToClipboard(color)}
                    >
                        {color.toUpperCase()}
                        <svg
                            xmlns="http://www.w3.org/2000/svg"
                            class="inline-block ml-1 w-4 h-4"
                            viewBox="0 0 24 24"
                            fill="none"
                            stroke="currentColor"
                            stroke-width="2"
                            stroke-linecap="round"
                            stroke-linejoin="round"
                        >
                            <path
                                d="M16 4h2a2 2 0 0 1 2 2v14a2 2 0 0 1-2 2H6a2 2 0 0 1-2-2V6a2 2 0 0 1 2-2h2"
                            ></path>
                            <rect x="8" y="2" width="8" height="4" rx="1" ry="1"
                            ></rect>
                        </svg>
                    </button>
                </div>
            {/each}
        </div>
    </div>

    
</div>

<div class="relative md:fixed bottom-0 left-0 right-0 mx-auto block text-center p-1">
    <p>@bagusindrayana</p>
</div>


